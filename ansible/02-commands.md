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

some command on servers
ansible all -m command -a "uptime" -i inventory
ansible all -m shell -a "df -h" -i inventory
ansible myservers -m ping -i inventory
ansible-playbook playbook.yml -i inventory 