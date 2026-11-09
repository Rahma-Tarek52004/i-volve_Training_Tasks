# Lab 26: Structured Configuration Management with Ansible Roles

## Objective
Convert the Lab 25 playbook into separate, reusable Ansible roles for **Docker**, **kubectl** and **Jenkins**, and run them from one playbook.

## Structure
```
Lab26-Ansible-Roles/
├── ansible.cfg
├── inventory.ini
├── lab26-playbook.yml
└── roles/
    ├── docker/
    │   ├── defaults/main.yml   # docker_packages
    │   └── tasks/main.yml
    ├── kubectl/
    │   ├── defaults/main.yml   # kubectl_version
    │   └── tasks/main.yml
    └── jenkins/
        ├── defaults/main.yml   # jenkins_key_url
        └── tasks/main.yml
```

## Playbook
```yaml
---
- name: Lab 26 - Configure managed node using roles
  hosts: managed
  become: true
  roles:
    - docker
    - kubectl
    - jenkins
```
`docker` runs before `jenkins` because the Jenkins role adds the `jenkins` user to the `docker` group.

## Why roles
- Tasks are split by responsibility and easy to reuse.
- Versions and URLs live in `defaults/main.yml`, so they can be overridden without editing tasks.

## Run
```bash
ansible-playbook lab26-playbook.yml --syntax-check
ansible-playbook lab26-playbook.yml -K
```

## Verify
```bash
find roles -type f
ansible managed -m shell -a "docker --version; kubectl version --client; java -version 2>&1 | head -1; systemctl is-active docker jenkins"
ansible managed -m shell -a "curl -sI http://localhost:8080/login | head -1; curl -sI http://localhost:8080/login | grep X-Jenkins:"
```
Result: `PLAY RECAP ok=11 changed=0 failed=0`. Everything was already installed by Lab 25, so `changed=0` also shows the roles are **idempotent**.

## Screenshots

**Roles playbook run, `changed=0`**
![Roles run](screenshots/01-roles-playbook-run-changed0.png)

**Roles tree, playbook, and verification**
![Roles tree and verification](screenshots/02-roles-tree-playbook-verification.png)

**Jenkins responds `HTTP/1.1 200 OK`**
![Jenkins HTTP 200](screenshots/03-jenkins-http-200.png)

**Final verification with a clean `curl` output**
![Final verification](screenshots/04-final-verification-clean-curl.png)

## Notes
- The Jenkins role uses Java 21 and the 2026 signing key (see the troubleshooting table in Lab 25).
