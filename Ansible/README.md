# Ansible Labs (24–28)

This section documents Ansible automation labs covering configuration management, playbooks, reusable roles, Ansible Vault, and AWS dynamic inventory.

## Labs Overview

| Lab | Topic | Description |
|---|---|---|
| [Lab 24](Lab24-Ansible-Initial-Config) | Ansible Configuration | Install Ansible, configure SSH key authentication, create inventory, and use ad-hoc commands to install Docker. |
| [Lab 25](Lab25-Ansible-Playbook) | Ansible Playbook | Automate the installation and configuration of Docker, kubectl, and Jenkins using a single playbook. |
| [Lab 26](Lab26-Ansible-Roles) | Ansible Roles | Refactor the playbook into reusable roles for Docker, kubectl, and Jenkins. |
| [Lab 27](Lab27-Ansible-MySQL-Vault) | MySQL Role & Ansible Vault | Create a reusable MySQL role, configure a database and user, and encrypt sensitive credentials using Ansible Vault. |
| [Lab 28](Lab28-Ansible-AWS-Dynamic-Inventory) | AWS Dynamic Inventory | Configure AWS EC2 dynamic inventory, discover instances using tags, and automate Nginx installation and configuration. |

---

## Lab 24 – Ansible Configuration

- Installed and configured Ansible on the control node.
- Generated an SSH key pair for passwordless authentication.
- Configured the managed node in the Ansible inventory.
- Tested connectivity using the Ansible ping module.
- Used ad-hoc commands to install and start Docker.
- Verified Docker installation and service status.

**Key concepts:** Ansible Installation, Control Node, Managed Node, SSH Authentication, Inventory, Ad-hoc Commands.

## Lab 25 – Ansible Playbook

- Created a YAML playbook to automate software installation.
- Installed Docker, kubectl, and Jenkins.
- Configured Docker and Jenkins services to start automatically.
- Used Ansible modules to manage packages, users, repositories, and services.
- Validated the playbook using syntax-check and execution commands.

**Key concepts:** YAML, Playbooks, Ansible Modules, Package Management, Service Management, Idempotency.

## Lab 26 – Ansible Roles

- Organized automation into reusable Ansible roles.
- Created separate roles for:
  - Docker
  - kubectl
  - Jenkins
- Used role defaults and task files to organize configurations.
- Created a `site.yml` playbook to execute all roles.
- Verified role execution and idempotency.

**Key concepts:** Ansible Roles, Role Structure, Defaults, Tasks, Reusability, Idempotency.

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

**Key concepts:** Ansible Roles, Variables, MySQL Modules, Database Management, Ansible Vault, Secrets Management.

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

**Key concepts:** AWS EC2, Dynamic Inventory, EC2 Tags, IAM Credentials, AWS SDK, Ansible Playbooks, Nginx Automation.

---

## Repository Structure

```text
Ansible/
│
├── Lab24-Ansible-Initial-Config/
│
├── Lab25-Ansible-Playbook/
│
├── Lab26-Ansible-Roles/
│
├── Lab27-Ansible-MySQL-Vault/
│
├── Lab28-Ansible-AWS-Dynamic-Inventory/
│   ├── screenshots/
│   ├── README.md
│   ├── aws-playbook.yml
│   └── aws_ec2.yml
│
├── Screenshots/
│
└── README.md
```

*Note: Each lab folder contains its own implementation files, documentation, and screenshots where applicable.*

## Technologies and Tools

- **Configuration Management:** Ansible
- **Operating System:** Ubuntu Linux
- **Automation Language:** YAML
- **Remote Access:** SSH
- **Containerization:** Docker
- **Container Orchestration:** Kubernetes (kubectl)
- **CI/CD:** Jenkins
- **Database:** MySQL
- **Secrets Management:** Ansible Vault
- **Cloud Platform:** AWS
- **Cloud Services:** EC2, IAM
- **Web Server:** Nginx

## Security Notes

- Never commit AWS access keys or secret access keys.
- Never commit an unencrypted Vault password or database password.
- Keep private SSH keys and AWS `.pem` files outside the repository.
- Use encrypted Vault files to protect sensitive variables.
- Use IAM permissions following the principle of least privilege.