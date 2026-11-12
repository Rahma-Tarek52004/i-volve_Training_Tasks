# Ansible Labs (24–28)

This section documents Ansible automation labs covering configuration management, playbooks, roles, Ansible Vault, and AWS dynamic inventory.

## Labs Overview

| Lab | Topic | Description |
|---|---|---|
| [Lab 24](Lab24-Ansible-Initial-Config) | Ansible Configuration | Install Ansible, configure SSH key authentication, create inventory, and use ad-hoc commands to install Docker. |
| [Lab 25](Lab25-Ansible-Playbook) | Ansible Playbook | Automate the installation and configuration of Docker, kubectl, and Jenkins using a single playbook. |
| [Lab 26](Lab26-Ansible-Roles) | Ansible Roles | Refactor the playbook into reusable roles for Docker, kubectl, and Jenkins. |
| [Lab 27](Lab27-Ansible-MySQL-Vault) | MySQL Role & Ansible Vault | Create a reusable MySQL role, configure a database and user, and encrypt sensitive credentials using Ansible Vault. |
| [Lab 28](Lab28) | AWS Dynamic Inventory | Configure AWS EC2 dynamic inventory, discover instances using tags, and automate Nginx installation and configuration. |

---

## Lab 24 – Ansible Configuration

- Installed and configured Ansible on the control node.
- Generated an SSH key pair for passwordless authentication.
- Configured the managed node in the Ansible inventory.
- Tested connectivity using the Ansible ping module.
- Used ad-hoc commands to install and start Docker.
- Verified Docker installation and service status.

## Lab 25 – Ansible Playbook

- Created a YAML playbook to automate software installation.
- Installed Docker, kubectl, and Jenkins.
- Configured Docker and Jenkins services to start automatically.
- Used Ansible modules to manage packages, users, repositories, and services.
- Validated the playbook using syntax-check and execution commands.

## Lab 26 – Ansible Roles

- Organized automation into reusable Ansible roles.
- Created separate roles for:
  - Docker
  - kubectl
  - Jenkins
- Used role defaults and task files to organize configurations.
- Created a `site.yml` playbook to execute all roles.
- Verified role execution and idempotency.

## Lab 27 – MySQL Role and Ansible Vault

- Created a reusable MySQL role.
- Installed and configured MySQL Server.
- Created an `ivolve` database.
- Created a database user named `ivolveuser`.
- Granted the user privileges on the `ivolve` database.
- Used the `community.mysql` collection to manage databases and users.
- Stored sensitive database credentials using Ansible Vault.
- Executed the MySQL playbook using `--ask-vault-pass`.
- Verified database access and listed databases using MySQL CLI.

**Key concepts:** Ansible Roles, Variables, MySQL Modules, Ansible Vault, Secrets Management.

## Lab 28 – AWS EC2 Dynamic Inventory

- Configured AWS credentials for Ansible.
- Installed the required AWS SDK dependencies and `amazon.aws` collection.
- Created an AWS EC2 dynamic inventory configuration.
- Used EC2 tags to dynamically discover running instances.
- Filtered instances using the tag `server=ivolve`.
- Displayed discovered hosts using `ansible-inventory`.
- Created an Ansible playbook to:
  - Install Nginx on the EC2 instance.
  - Create a custom `index.html` page.
  - Start and enable the Nginx service.
- Executed the playbook against dynamically discovered EC2 instances.
- Verified the Nginx installation and web page.

**Key concepts:** AWS EC2, Dynamic Inventory, EC2 Tags, IAM Credentials, Ansible Playbooks, Nginx Automation.

---

## Repository Structure

```text
Ansible/
│
├── screenshots/
├── Lab28/
│   ├── screenshots/
│   ├── README.md
│   ├── aws-playbook.yml
│   └── aws_ec2.yml
│
├── roles/
│   ├── docker/
│   ├── kubectl/
│   ├── jenkins/
│   └── mysql/
│
├── mysql.yml
├── main.yml
├── vars.yml
├── vault-EXAMPLE.yml
└── README.md
```

*Note: Update the structure if your actual folders or filenames differ.*

## Technologies and Tools

- Ansible
- YAML
- Ubuntu Linux
- SSH
- Docker
- Kubernetes kubectl
- Jenkins
- MySQL
- Ansible Vault
- AWS EC2
- AWS IAM
- Nginx

## Security Notes

- Never commit AWS access keys or secret access keys.
- Never commit an unencrypted Vault password or database password.
- Keep private SSH keys and AWS `.pem` files outside the repository.
- Use encrypted Vault files for sensitive variables.