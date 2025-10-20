# Structured Configuration Management with Ansible Roles

This repository demonstrates how to use **Ansible roles** to manage complex software installations on a managed node. It covers installing and configuring **Docker**, **kubectl**, and **Jenkins** using reusable roles.

---

## Files and Directories

- **`inventory`**
```ini
[node1]
192.168.1.32 ansible_python_interpreter=/usr/local/bin/python3.9
```

- **`ansible.cfg`**
```ini
[defaults]
inventory = ./inventory
remote_user = ali
host_key_checking = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

- **Roles Directory**
```
roles/
├── docker/
│   └── tasks/
│       └── main.yml
├── jenkins/
│   └── tasks/
│       └── main.yml
└── kubectl/
    └── tasks/
        └── main.yml
```

Each role contains tasks to install and configure the corresponding software:

- **Docker Role**
  - Removes Podman and related tools
  - Installs Docker Engine and CLI
  - Enables and starts Docker service
  - Displays Docker version

- **Jenkins Role**
  - Installs Java 17
  - Adds Jenkins repository and GPG key
  - Installs Jenkins
  - Enables and starts Jenkins service
  - Reads and displays the initial admin password

- **kubectl Role**
  - Downloads latest `kubectl` binary
  - Ensures executable permissions
  - Verifies installation

---

## Playbook

- **`site.yml`** (or `devops_tools.yml`)
```yaml
- name: Configure DevOps tools
  hosts: node1
  become: yes
  roles:
    - docker
    - kubectl
    - jenkins
```

---

## Prerequisites

- Ansible installed on the control node.
- SSH access to `ali@192.168.1.32`.
- Sudo privileges for software installation.
- Python 3.9 installed on the managed node (or adjust `ansible_python_interpreter`).
- Optional: passwordless SSH key for automation.

---

## Steps to Run

### 1. Verify connectivity
```bash
ansible node1 -m ping
```
Expected output:
```
192.168.1.32 | SUCCESS => { "changed": false, "ping": "pong" }
```

### 2. Run the playbook
```bash
ansible-playbook -i ./inventory site.yml
```
- This will execute all roles: Docker → kubectl → Jenkins.

---

## Verification

- **Docker**
```bash
ansible node1 -m command -a "docker --version"
```

- **kubectl**
```bash
ansible node1 -m command -a "/usr/local/bin/kubectl version --client"
```

- **Jenkins**
```bash
ansible node1 -m shell -a "systemctl status jenkins"
```
- The playbook also prints the **initial Jenkins admin password**.

---

## Notes

- Roles use **`command`**, **`get_url`**, and **`file`** modules. For idempotency, consider replacing `command` installs with package modules (`dnf`, `apt`) where appropriate.
- `become: yes` ensures tasks requiring root privileges are executed safely.
- `ignore_errors: yes` is used in Jenkins service start to avoid playbook interruption if already running.

---

## Optional Improvements

- Add **handlers** to restart services only when configurations change.
- Use `vars/main.yml` in roles to parameterize software versions.
- Add **group_vars** or **host_vars** for multiple managed nodes.

---

This README guides you through **structured Ansible role-based automation** for Docker, kubectl, and Jenkins, including running the playbook and verifying installations.  


