# Automated Web Server Configuration Using Ansible Playbooks

This repository demonstrates how to automate the configuration of a web server on a managed node using Ansible. The playbook installs Nginx, customizes the default web page, and ensures the service is running.

---

## Files

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

- **`webserver.yml`** (Ansible playbook)
```yaml
- name: Configure a web server
  hosts: node1
  become: yes
  tasks:
    - name: Install Nginx
      command: dnf install -y nginx

    - name: Enable and start Nginx
      systemd:
        name: nginx
        enabled: yes
        state: started

    - name: Customize the default web page
      copy:
        dest: /usr/share/nginx/html/index.html
        co

