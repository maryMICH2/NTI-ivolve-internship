# Initial Ansible Setup and Ad-Hoc Execution

This repository demonstrates the initial configuration of Ansible on a control node, adding a managed node, and performing basic ad-hoc commands.

---

## Files

* **`inventory`**

```ini
[node1]
192.168.1.32 ansible_python_interpreter=/usr/local/bin/python3.9
```

* Contains the host (`node1`) with its Python interpreter path.
* Optionally, specify the private key if you created a dedicated key for Ansible:

```ini
192.168.1.32 ansible_python_interpreter=/usr/local/bin/python3.9 ansible_ssh_private_key_file=~/.ssh/id_rsa_ansible
```

* **`ansible.cfg`**

```ini
[defaults]
inventory = ./inventory
remote_user = ali
host_key_checking = False

[privilege_escalation]
become = True
become_method = sudo
become_user =
```

