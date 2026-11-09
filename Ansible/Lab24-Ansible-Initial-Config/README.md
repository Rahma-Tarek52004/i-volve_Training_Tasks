# Lab 24: Initial Ansible Configuration and Ad-Hoc Execution

## Objective
Install and configure Ansible on a control node, generate a new SSH key, copy it to a managed node with `ssh-copy-id`, create an inventory, and run an ad-hoc command that installs Docker.

## Environment
- Control node: WSL2 (Ubuntu) on Windows, Ansible installed here
- Managed node: `worker-1` (VMware VM, Ubuntu 22.04, `192.168.247.131`, user `k8s`), also a Kubernetes v1.28.15 worker from Lab 9
- `worker-1` already runs `containerd.io` (the cluster runtime) and has about 2.4 GB of RAM
- `master-1` (`192.168.247.130`) is not used in this lab

## Steps

### 1. Install Ansible on the control node
```bash
sudo apt update
sudo apt install -y ansible
ansible --version
```

### 2. Generate a new SSH key
```bash
ssh-keygen -t ed25519 -C "ansible-control"
```

### 3. Copy the public key to the managed node
```bash
ssh-copy-id k8s@192.168.247.131
```

### 4. Create the inventory and the config file (`inventory.ini`, `ansible.cfg`)
```ini
[managed]
worker-1 ansible_host=192.168.247.131 ansible_user=k8s
```
```ini
[defaults]
inventory = inventory.ini
interpreter_python = /usr/bin/python3
host_key_checking = False
```
- `interpreter_python` removes the "discovered Python interpreter" warning
- `host_key_checking = False` avoids the fingerprint prompt on the lab VM

### 5. Test the connection
```bash
ansible managed -m ping
```
The result is `pong` from `worker-1`.

### 6. Check what is already installed before changing anything
```bash
ansible managed -i inventory.ini -m shell -a "which docker containerd; dpkg -l | grep -E 'docker|containerd' | awk '{print \$2, \$3}'"
```
The worker has `containerd.io 2.4.2` from the Docker repository and no `docker` command.

### 7. Simulate the install
```bash
ansible managed -m shell -a "apt-get -s install docker-ce docker-ce-cli | grep -E '^(Inst|Remv)'"
```
Only `Inst` lines appear (`docker-ce`, `docker-ce-cli`, `docker-buildx-plugin`, `docker-compose-plugin`, `docker-ce-rootless-extras`, `pigz`) and no `Remv` line, so nothing will be removed.

### 8. Ad-hoc command to install Docker
```bash
ansible managed -b -K -m apt -a "name=docker-ce state=present update_cache=yes"
```
- `-b` runs the task with sudo (become)
- `-K` asks for the sudo password of `k8s` on the worker
- The result shows `6 newly installed, 0 to remove`

### 9. Verify
```bash
ansible managed -m command -a "docker --version"
ansible managed -m shell -a "systemctl is-active docker containerd"
```
Docker is `29.9.3` and both `docker` and `containerd` are `active`.

## Screenshots
<!-- Add these too: ansible --version, ssh-copy-id output, inventory.ini, ping pong -->

Simulation (only `Inst`, no `Remv`) and the ad-hoc Docker install

![Simulation and install](screenshots/01-adhoc-simulation-and-docker-install.png)

Docker version and services active

![Docker verification](screenshots/02-docker-version-and-services-active.png)

## Notes and issues
- **Wrong package avoided:** the first suggestion was `docker.io`. It pulls its own `containerd` and `runc` from Ubuntu and can conflict with or remove `containerd.io`, which would take the Kubernetes worker down. I stopped that command with Ctrl+C at the `BECOME password:` prompt before typing the password, checked the installed packages (step 6), and used `docker-ce` instead, which works with the existing `containerd.io`.
- **Simulation first:** `apt-get -s` only prints what would happen. Checking for `Remv` lines proves the cluster runtime is safe before the real install.
- **`Connection timed out` on port 22:** at one point `worker-1` did not answer SSH. The VM was off or overloaded. Powering it on, waiting two minutes, and checking with `ping` and `ansible managed -m ping` fixed it.
- **`BECOME password:` prompt:** it is the sudo password of the `k8s` user on the worker. Nothing is shown while typing, and pressing Ctrl+C cancels the run.
- **`NEEDRESTART-*` lines:** they only say some services could be restarted for the new kernel. They are informational. I did not reboot the worker because it is part of the cluster.
- **Python interpreter warning:** fixed by `interpreter_python = /usr/bin/python3` in `ansible.cfg`.
- **Memory:** the worker has about 2.4 GB of RAM, so I closed other VMs and heavy apps while working.
- **Secrets:** the sudo password and the private key are never stored in this repository.

## Conclusion
Ansible on WSL controls `worker-1` over key-based SSH, the inventory and `ansible.cfg` are in place, and Docker was installed with a single ad-hoc command without touching the cluster's `containerd.io`.