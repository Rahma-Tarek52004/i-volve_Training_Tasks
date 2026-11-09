# Lab 25: Automated Configuration Using Ansible Playbooks

## Objective
Write one Ansible playbook that installs Docker, kubectl and Jenkins on the managed node, then verify the configuration.

## Environment
- Control node: WSL2 (Ubuntu), Ansible from Lab 24
- Managed node: `worker-1` (`192.168.247.131`, Ubuntu 22.04, about 2.4 GB RAM), also a Kubernetes v1.28.15 worker
- Docker was already installed on the worker in Lab 24
- This Jenkins runs on the worker VM and is separate from the Jenkins container on my laptop (`localhost:8080`), so there is no port conflict

## Files
| File | Purpose |
|------|---------|
| `lab25-playbook.yml` | Docker, kubectl and Jenkins in one playbook |
| `inventory.ini` | Managed node `worker-1` |
| `ansible.cfg` | Inventory path, Python interpreter, host key checking |

## Steps

### 1. Write the playbook (`lab25-playbook.yml`)
The playbook has three parts, all with `hosts: managed` and `become: true`:

| Part | What it does |
|------|--------------|
| Docker | Adds the Docker apt repo only if it is missing, installs `docker-ce`, CLI, buildx and compose plugins, enables and starts the service |
| kubectl | Downloads `v1.28.15` (same as the cluster) to `/usr/local/bin/kubectl` |
| Jenkins | Installs Java 21, adds the Jenkins key and repo, installs Jenkins, adds the `jenkins` user to the `docker` group, enables and starts the service |

```bash
ansible-playbook lab25-playbook.yml --syntax-check
ansible-playbook lab25-playbook.yml -K
```

### 2. Expected result
- Docker tasks are `ok` (already installed) and the repo tasks are `skipping`
- kubectl and Jenkins tasks are `changed` on the first run
- `PLAY RECAP` shows `failed=0 unreachable=0`

### 3. Verify on the managed node
```bash
ansible managed -m command -a "docker --version"
ansible managed -m command -a "kubectl version --client"
ansible managed -m shell -a "java -version 2>&1 | head -1; systemctl is-active docker jenkins"
ansible managed -m command -a "curl -sI http://localhost:8080/login"
```
| Check | Result |
|-------|--------|
| Docker | `29.9.3` |
| kubectl | `v1.28.15` |
| Java | `openjdk 21.0.13` |
| `docker` and `jenkins` services | `active` |
| Jenkins `/login` | `HTTP/1.1 200 OK`, `X-Jenkins: 2.580.2` |

### 4. Open Jenkins and finish the setup wizard
- The first admin password is read on the worker with `ansible managed -b -K -m command -a "cat /var/lib/jenkins/secrets/initialAdminPassword"`
- Open `http://192.168.247.131:8080`, unlock with that password, install the suggested plugins, create the first admin user
- The password and the admin account are not stored in this repository

## Screenshots

Playbook run and PLAY RECAP (`failed=0`)

![Playbook recap](screenshots/01-playbook-run-recap.png)

Verification of Docker, kubectl, Java and Jenkins

![Verification](screenshots/02-verification-docker-kubectl-java-jenkins.png)

Jenkins setup wizard on the managed node

![Setup wizard](screenshots/03-jenkins-setup-wizard.png)

Create first admin user (credentials blurred)

![Admin user](screenshots/04-jenkins-create-admin-BLURRED.png)

Jenkins dashboard

![Dashboard](screenshots/05-jenkins-dashboard.png)

## Notes and issues
- **Playbook seemed stuck on `Jenkins | prerequisites`:** downloading Java on a 2.4 GB VM is slow and prints nothing. Even a simple `ansible` command timed out, which showed the VM was starved of memory. Closing other VMs and apps and waiting solved it. Re-running the playbook is safe because finished tasks return `ok`.
- **`NO_PUBKEY 7198F4B714ABFC68` and "repository is not signed":** the key in the playbook (`jenkins.io-2023.key`) had expired (`gpg --show-keys` showed state `e` for key `5BA31D57EF5975CA`). Jenkins now signs with a new key. I changed the URL to `jenkins.io-2026.key`, and verified that key contains `7198F4B714ABFC68`.
- **The key task stayed `ok` after the URL change:** `get_url` does not replace a file that already exists, so the old expired key stayed on the worker. `force: true` replaces it once. After the new key was in place the recap showed it as `ok` again.
- **`jenkins.service` failed to start:** `journalctl -u jenkins` showed "Running with Java 17 ... older than the minimum required version (Java 21)". The playbook installed `openjdk-17-jre`. I changed it to `openjdk-21-jre`.
- **Removing Java 17 pulled in Java 11:** `apt` installed `openjdk-11-jre-headless` and `default-jre-headless` because `ca-certificates-java` needs a Java runtime. Java 21 was installed afterwards, and the final `java -version` shows 21.
- **`Start request repeated too quickly`:** systemd stops retrying after several failures. `systemctl reset-failed jenkins` clears the counter before re-running the playbook.
- **`Connection timed out during banner exchange`:** SSH answered slowly because the worker was low on memory. Suspending unused VMs and stopping Jenkins when not needed helped.
- **Idempotency:** after the successful run, running the playbook again should show `changed=0`. The first successful recap showed `changed=2` because Java and the Jenkins service were changed in that run.
- **Two Jenkins instances:** `localhost:8080` is the Jenkins container on my laptop and `192.168.247.131:8080` is the new one on the worker. They are on different machines, so ports do not conflict.
- **Worker RAM:** Jenkins competes with the kubelet and pods. After taking the screenshots I stop it with `ansible managed -b -K -m service -a "name=jenkins state=stopped"`.

## Conclusion
One playbook installs and configures Docker, kubectl and Jenkins on `worker-1`. The Jenkins repository key and the Java version problems were found from the error messages and fixed, and Jenkins runs and answers on port 8080.