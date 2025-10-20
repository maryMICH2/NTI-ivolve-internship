# Securing Sensitive Data with Ansible Vault

## Project Overview

This repository contains Ansible playbooks for automating MySQL installation and database setup on a managed node.
It demonstrates the use of:

* Inventory files
* Configuration via `ansible.cfg`
* Ansible Vault for encrypting sensitive data
* Command-based playbooks for database operations

---

## Project Structure

```
NTI-ivolve-internship/
├── ansible.cfg
├── inventory
├── playbook.yml
├── vault.yaml
└── README.md
```

---

### Configuration File: ansible.cfg

```ini
[defaults]
inventory= ./inventory
remote_user= ali
host_key_checking = False

[privilege_escalation] 
become = True
```

* `inventory`: points to the hosts inventory file
* `remote_user`: user used to connect to managed nodes
* `host_key_checking`: disabled for automatic SSH trust
* `become`: enables privilege escalation for tasks requiring root access

---

### Inventory File: inventory

```ini
[node1]
192.168.1.32 ansible_python_interpreter=/usr/local/bin/python3.9
```

* Defines the managed node `node1`
* Sets Python 3.9 as the interpreter on the managed host

---

### Playbook: playbook.yml

```yaml
- name: Setup MySQL and iVolve DB (command-based)
  hosts: node1
  become: yes
  vars_files:
    - vault.yaml

  tasks:
    - name: Install MySQL server
      ansible.builtin.command: yum install -y @mysql
      become: yes

    - name: Start MySQL service
      ansible.builtin.command: systemctl enable --now mysqld
      become: yes

    - name: Create iVolve database
      ansible.builtin.command: >
        mysql -u root -e "CREATE DATABASE IF NOT EXISTS iVolve;"
      become: yes

    - name: Create MySQL user 'ali'
      ansible.builtin.command: >
        mysql -u root -e
        "CREATE USER IF NOT EXISTS 'ali'@'localhost' IDENTIFIED BY '{{ mysql_user_password }}';"
      become: yes

    - name: Grant privileges to MySQL user 'ali'
      ansible.builtin.command: >
        mysql -u root -e
        "GRANT ALL PRIVILEGES ON iVolve.* TO 'ali'@'localhost' WITH GRANT OPTION;"
      become: yes

    - name: Validate DB by listing databases as ali
      ansible.builtin.shell: "mysql -u ali -p{{ mysql_user_password }} -e 'SHOW DATABASES;'"
      register: db_list

    - name: Show databases
      ansible.builtin.debug:
        var: db_list.stdout_lines
```

* Uses `vars_files` to include encrypted credentials from `vault.yaml`
* Installs MySQL, creates a database and user, grants privileges, and validates DB access

---

### Vault File: vault.yaml

* Contains sensitive information (like database user passwords) encrypted with Ansible Vault.
* Example structure (encrypted in practice):

```yaml
mysql_user_password: admin_password_here
```

* Encrypt the file using:

```bash
ansible-vault encrypt vault.yaml
```

---

## Usage Instructions

1. Ensure Python and Ansible are installed on the control node.
2. Update inventory with your managed node IP and Python interpreter path.
3. Update `vault.yaml` with desired MySQL user passwords.
4. Run the playbook:

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

5. Check output to verify that MySQL, database, and user have been correctly created.

---

## Notes

* Always use **Ansible Vault** for sensitive information like passwords.
* If you modify the playbook, stage and commit changes to Git before pushing to GitHub:

```bash
git add .
git commit -m "Update playbook or vault"
git push origin main
```

* Git does not track empty folders. Add a placeholder file like `.gitkeep` if needed.
* Use `git status` often to monitor uncommitted or conflicted files.

---


