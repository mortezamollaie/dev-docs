# ⚡ Ansible Commands & Playbooks

Quick reference for ad-hoc commands, playbooks, output colors, and idempotency.

---

## 🏓 Ad-Hoc Command: `ping`

A single Ansible command to perform a task quickly from the command line. This is the most basic operation you can run.

> ⚠️ Ansible `ping` is **not** ICMP ping — it tests whether Ansible can connect and run a Python module on the host.

### Basic Syntax

```bash
ansible all -m ping
```

| Part | Meaning |
|------|---------|
| `all` | Target group or host pattern from the inventory |
| `-m ping` | Module to run (`ping`) |

### Sample Output

```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web2 | UNREACHABLE! => ...
```

---

## 🛠️ Common Ad-Hoc Commands

### Check Disk Usage on All Hosts

```bash
ansible all -m shell -a "df -h"
```

### Run a Command on the `web` Group

```bash
ansible web -m command -a "uptime"
```

### Collect System Facts for a Single Host

```bash
ansible web1 -m setup
```

### Run with a Custom Inventory File

```bash
ansible all -m ping -i inventory
```

---

## 📜 Ansible Playbook

A **playbook** is a YAML file that defines one or more **plays**. Each play runs a set of **tasks** on selected hosts.

| Term | Description |
|------|-------------|
| **Playbook** | YAML file containing automation steps |
| **Play** | A set of tasks run on a group of hosts |
| **Task** | A single action performed on a host |

### Common Task Types
- Execute a command
- Run a script
- Install a package
- Shutdown / restart a service

### Sample Playbook

```yaml
# site.yml
---
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Ensure httpd is installed
      ansible.builtin.package:
        name: httpd
        state: present

    - name: Ensure httpd is running
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Deploy index page
      ansible.builtin.copy:
        content: "Hello from Ansible!"
        dest: /var/www/html/index.html
```

### Run a Playbook

```bash
ansible-playbook site.yml -i inventory
```

### Dry Run (Check Mode)

```bash
ansible-playbook site.yml -i inventory --check
```

---

## 🎨 Ansible Output Colors

| Color | Meaning |
|-------|---------|
| 🟢 **Green** | Task ran successfully — no change was made |
| 🟡 **Yellow** | Task ran successfully — a change was made |
| 🔴 **Red** | Task failed |
| 🔵 **Blue** | Task was skipped (condition not met) |

---

## 🔁 Idempotency

Tasks only make changes when the current system state differs from the desired state.

**Benefits:** security, predictability, repeatability.

### Example

```yaml
- name: Ensure httpd is started
  ansible.builtin.service:
    name: httpd
    state: started
```

| Current State | Result |
|---------------|--------|
| `httpd` is **not** running | Ansible starts it ✅ |
| `httpd` is **already** running | Ansible does nothing ✅ |

> 💡 You can run the same playbook multiple times safely — Ansible only applies what is needed.

---

## 🚀 Quick Reference — Commands on Servers

Every ad-hoc command below uses `-i inventory` to point at your inventory file.

### Test Connectivity

```bash
ansible myservers -m ping -i inventory
```

### Check Uptime on All Hosts

```bash
ansible all -m command -a "uptime" -i inventory
```

### Check Disk Usage on All Hosts

```bash
ansible all -m shell -a "df -h" -i inventory
```

### Run a Playbook

```bash
ansible-playbook playbook.yml -i inventory
```

---

## 🎯 Useful Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Inventory file | `ansible all -m ping -i inventory` |
| `--limit` | Run on a subset of hosts | `ansible all -m ping -i inventory --limit web1` |
| `-u` | Remote user | `ansible all -m ping -i inventory -u root` |
| `--become` / `-b` | Run with sudo | `ansible web -m command -a "whoami" -b` |
| `-K` | Prompt for sudo password | `ansible web -m command -a "whoami" -b -K` |
| `-v` | Verbose output (use `-vvv` for debug) | `ansible all -m ping -i inventory -v` |
| `--check` | Dry run (no changes) | `ansible-playbook site.yml -i inventory --check` |
| `--diff` | Show file diffs | `ansible-playbook site.yml -i inventory --diff` |

---

## 📦 More Ad-Hoc Samples

### Install a Package (RHEL/CentOS)

```bash
ansible web -m yum -a "name=httpd state=present" -b -i inventory
```

### Install a Package (Debian/Ubuntu)

```bash
ansible web -m apt -a "name=nginx state=present" -b -i inventory
```

### Restart a Service

```bash
ansible web -m service -a "name=httpd state=restarted" -b -i inventory
```

### Copy a File to All Hosts

```bash
ansible all -m copy -a "src=./config.conf dest=/etc/app/config.conf" -b -i inventory
```

### Create a Directory

```bash
ansible web -m file -a "path=/opt/myapp state=directory mode=0755" -b -i inventory
```

### Reboot Hosts (with confirmation)

```bash
ansible web -m reboot -a "reboot_timeout=300" -b -i inventory
```

---

## 📜 More Playbook Samples

### Playbook with Variables

```yaml
# deploy.yml
---
- name: Deploy application
  hosts: web
  become: true
  vars:
    app_port: 8080
    app_user: deploy

  tasks:
    - name: Create app directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        owner: "{{ app_user }}"
        mode: "0755"

    - name: Open firewall port
      ansible.builtin.firewalld:
        port: "{{ app_port }}/tcp"
        permanent: true
        state: enabled
```

### Playbook with Handlers

Handlers run only when a task notifies them (e.g. after a config change).

```yaml
# nginx.yml
---
- name: Configure Nginx
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Deploy Nginx config
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

### Playbook with Conditionals

```yaml
# patch.yml
---
- name: Patch servers
  hosts: all
  become: true

  tasks:
    - name: Update packages on RedHat
      ansible.builtin.yum:
        name: "*"
        state: latest
      when: ansible_os_family == "RedHat"

    - name: Update packages on Debian
      ansible.builtin.apt:
        upgrade: dist
      when: ansible_os_family == "Debian"
```

### Run Specific Tags Only

```yaml
# site.yml (with tags)
---
- name: Full server setup
  hosts: web
  become: true

  tasks:
    - name: Install packages
      ansible.builtin.package:
        name: httpd
        state: present
      tags: [install]

    - name: Configure firewall
      ansible.builtin.firewalld:
        service: http
        permanent: true
        state: enabled
      tags: [firewall]
```

```bash
# Run only tasks tagged "install"
ansible-playbook site.yml -i inventory --tags install

# Skip firewall tasks
ansible-playbook site.yml -i inventory --skip-tags firewall
```

---

## 🔧 Other Useful Commands

```bash
# List all hosts in inventory
ansible all -i inventory --list-hosts

# List hosts in a specific group
ansible web -i inventory --list-hosts

# Show gathered facts for a host
ansible web1 -m setup -i inventory

# Filter facts (e.g. only memory info)
ansible web1 -m setup -a "filter=ansible_memory*" -i inventory

# Syntax-check a playbook
ansible-playbook site.yml --syntax-check

# List tasks in a playbook without running them
ansible-playbook site.yml -i inventory --list-tasks
``` 