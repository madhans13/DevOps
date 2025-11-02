# 🚀 Ansible Interview Preparation Guide

A comprehensive guide to prepare for Ansible-related DevOps interviews, covering core concepts, commands, and real-world scenarios.

---

## 📋 Table of Contents

- [What is Ansible?](#what-is-ansible)
- [Core Concepts](#core-concepts)
- [Inventory Management](#inventory-management)
- [Ad-hoc Commands](#ad-hoc-commands)
- [Playbooks](#playbooks)
- [Variables & Templates](#variables--templates)
- [Handlers](#handlers)
- [Roles](#roles)
- [Ansible Galaxy](#ansible-galaxy)
- [Common Interview Questions](#common-interview-questions)
- [Useful Commands](#useful-commands)
- [Best Practices](#best-practices)

---

## 🎯 What is Ansible?

Ansible is a powerful **Configuration Management & Automation tool** with the following characteristics:

- **Agentless Architecture**: Works over SSH, no agent installation required on managed nodes
- **Written in Python**: Easy to extend and customize
- **YAML-based**: Uses simple, human-readable YAML syntax for playbooks
- **Idempotent**: Running the same playbook multiple times produces the same result

---

## 🧩 Core Concepts

| Term | Description |
|------|-------------|
| **Control Node** | The machine where Ansible is installed and run from |
| **Managed Node** | Remote systems that Ansible manages (e.g., EC2 instances) |
| **Inventory** | File containing a list of target hosts |
| **Module** | Small code units that perform specific tasks (apt, copy, service, etc.) |
| **Task** | A single action within a playbook |
| **Play** | A group of tasks executed on specific hosts |
| **Role** | Structured, reusable set of tasks, variables, and templates |
| **Facts** | System information automatically gathered from managed nodes |

---

## 🖥️ Inventory Management

### Default Inventory Location
```
/etc/ansible/hosts
```

### Custom Inventory Example (`inventory.ini`)

```ini
[web]
13.201.21.114 ansible_user=ubuntu ansible_ssh_private_key_file=~/Downloads/mykey.pem

[db]
15.206.174.20 ansible_user=ubuntu ansible_ssh_private_key_file=~/Downloads/mykey.pem

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Test Connection

```bash
ansible all -i inventory.ini -m ping
```

---

## ⚡ Ad-hoc Commands

Quick one-liner commands for immediate tasks:

```bash
# Check connectivity
ansible all -m ping

# Execute shell commands
ansible all -m command -a "uptime"

# Install packages
ansible web -b -m apt -a "name=nginx state=present update_cache=yes"

# Copy files
ansible web -b -m copy -a "src=index.html dest=/var/www/html/index.html"

# Manage services
ansible web -b -m service -a "name=nginx state=restarted"

# Gather facts
ansible all -m setup
```

**Flag Reference:**
- `-m`: Specify module
- `-a`: Module arguments
- `-b`: Become (use sudo)
- `-i`: Inventory file path

---

## 📘 Playbooks

Playbooks are YAML files that define automation workflows.

### Basic Playbook Structure

```yaml
---
- name: Deploy web server
  hosts: web
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Copy HTML file
      copy:
        src: index.html
        dest: /var/www/html/index.html
    
    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

### Run Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml

# Dry-run mode
ansible-playbook playbook.yml --check

# Syntax check
ansible-playbook playbook.yml --syntax-check
```

---

## 🏗️ Variables & Templates

### Using Variables

```yaml
---
- name: Install package
  hosts: web
  become: yes
  vars:
    pkg_name: nginx
    html_src: index.html
  
  tasks:
    - name: Install {{ pkg_name }}
      apt:
        name: "{{ pkg_name }}"
        state: present
```

### Jinja2 Templates

**Template File:** `templates/index.html.j2`
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ site_title }}</title>
</head>
<body>
    <h1>Welcome to {{ ansible_hostname }}</h1>
    <p>Environment: {{ environment }}</p>
</body>
</html>
```

**Playbook Task:**
```yaml
- name: Deploy template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
```

---

## ♻️ Handlers

Handlers are triggered only when notified and run once at the end of a play.

```yaml
tasks:
  - name: Update nginx configuration
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

---

## 🧱 Roles

Roles provide a structured way to organize playbooks for reusability.

### Create Role Structure

```bash
ansible-galaxy init webserver
```

### Directory Structure

```
roles/webserver/
├── tasks/main.yml          # Main task list
├── handlers/main.yml       # Handler definitions
├── templates/              # Jinja2 templates
├── files/                  # Static files
├── vars/main.yml           # Variables
├── defaults/main.yml       # Default variables
└── meta/main.yml           # Role metadata
```

### Example Role Task (`roles/webserver/tasks/main.yml`)

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Copy HTML file
  copy:
    src: index.html
    dest: /var/www/html/index.html

- name: Start nginx service
  service:
    name: nginx
    state: started
    enabled: yes
```

### Use Role in Playbook

```yaml
---
- name: Deploy website using roles
  hosts: web
  become: yes
  roles:
    - webserver
```

---

## 🌐 Ansible Galaxy

Ansible Galaxy is a repository for community-contributed roles.

### Install Role from Galaxy

```bash
ansible-galaxy install geerlingguy.nginx
```

### List Installed Roles

```bash
ansible-galaxy list
```

### Use Galaxy Role

```yaml
---
- name: Deploy nginx using Galaxy role
  hosts: web
  become: yes
  roles:
    - geerlingguy.nginx
```

**Browse roles:** [https://galaxy.ansible.com](https://galaxy.ansible.com)

---

## 💡 Common Interview Questions

### Conceptual Questions

1. **What makes Ansible agentless?**
   - Ansible uses SSH (Linux) or WinRM (Windows) to communicate with nodes without requiring agent installation.

2. **Explain idempotency in Ansible.**
   - Running the same playbook multiple times produces the same result without causing unintended side effects.

3. **What is the difference between a playbook and a role?**
   - A playbook is a YAML file with tasks. A role is an organized structure of tasks, variables, templates, and handlers for reusability.

4. **What are Ansible facts?**
   - System information automatically gathered from managed nodes (OS, IP, hostname, etc.). Access with `ansible_facts`.

5. **Handlers vs Tasks?**
   - Tasks run sequentially. Handlers run only when notified and execute once at the end of a play.

### Practical Questions

6. **How do you debug a failing playbook?**
   ```bash
   ansible-playbook playbook.yml -vvv
   ansible-playbook playbook.yml --check
   ```

7. **How to run only specific tasks?**
   ```bash
   # Using tags
   ansible-playbook playbook.yml --tags "install"
   ansible-playbook playbook.yml --skip-tags "copy"
   ```

8. **How to handle secrets in Ansible?**
   - Use **Ansible Vault** to encrypt sensitive data:
   ```bash
   ansible-vault create secrets.yml
   ansible-vault encrypt vars.yml
   ansible-playbook playbook.yml --ask-vault-pass
   ```

9. **What is the difference between `copy` and `template` modules?**
   - `copy`: Transfers static files as-is
   - `template`: Processes Jinja2 templates with variables before copying

10. **How to limit playbook execution to specific hosts?**
    ```bash
    ansible-playbook playbook.yml --limit web1.example.com
    ```

---

## 🧮 Useful Commands

| Command | Description |
|---------|-------------|
| `ansible --version` | Check Ansible version |
| `ansible-inventory --list -i inventory.ini` | View parsed inventory |
| `ansible all -m ping` | Test connectivity to all hosts |
| `ansible-playbook playbook.yml --check` | Dry-run mode (no changes) |
| `ansible-playbook playbook.yml --syntax-check` | Validate YAML syntax |
| `ansible-galaxy init role_name` | Create role skeleton |
| `ansible-galaxy install role_name` | Install role from Galaxy |
| `ansible-doc module_name` | View module documentation |
| `ansible all -m setup` | Gather facts from all hosts |

---

## ✅ Best Practices

1. **Use Version Control**: Store playbooks and roles in Git
2. **Idempotency**: Ensure tasks can run multiple times safely
3. **Use Roles**: Organize code into reusable roles
4. **Naming Conventions**: Use descriptive task and variable names
5. **Ansible Vault**: Encrypt sensitive data (passwords, keys)
6. **Tags**: Use tags for selective task execution
7. **Testing**: Always test with `--check` before production runs
8. **Documentation**: Comment complex tasks and maintain README files
9. **Minimize Privileges**: Use `become` only when necessary
10. **Dynamic Inventory**: Use cloud provider APIs for dynamic infrastructure

---

## 🔧 Debugging Tips

```bash
# Verbose output
ansible-playbook playbook.yml -v    # verbose
ansible-playbook playbook.yml -vvv  # very verbose

# Syntax validation
ansible-playbook playbook.yml --syntax-check

# Step-by-step execution
ansible-playbook playbook.yml --step

# Start at specific task
ansible-playbook playbook.yml --start-at-task="Install nginx"
```

---

## 📚 Additional Resources

- [Official Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible GitHub Repository](https://github.com/ansible/ansible)
- [Best Practices Guide](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

---

## 🤝 Contributing

Feel free to submit issues or pull requests to improve this guide!

---

## 📄 License

This guide is open-source and available for educational purposes.

---

**Good luck with your interviews! 🚀**
