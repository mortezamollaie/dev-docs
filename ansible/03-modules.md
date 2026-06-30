# 🧩 Ansible Modules

Modules are the units of work Ansible executes on managed hosts. Each module handles a specific task — install a package, manage a service, copy a file, etc.

---

## 📂 Module Categories

| Category | Examples |
|----------|----------|
| 🖥️ **System** | `selinux`, `service`, `ufw`, `timezone`, `firewalld` |
| ⌨️ **Commands** | `command`, `script`, `shell`, `raw` |
| 📁 **Files** | `copy`, `archive`, `find`, `file`, `template`, `lineinfile` |
| 🗄️ **Database** | `mongodb`, `mysql`, `postgresql` |
| ☁️ **Cloud** | `aws_*`, `azure_*`, `gcp_*`, `docker_*` |
| 🪟 **Windows** | `win_copy`, `win_file`, `win_service` |

> 💡 Prefer dedicated modules (`yum`, `service`, `file`) over `command`/`shell` — they are idempotent and safer.

---

## 📦 Package Management

### Install httpd (RHEL/CentOS)

```yaml
---
- name: Install Apache
  hosts: all
  become: true

  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present
```

### Ad-Hoc Equivalent

```bash
ansible all -m yum -a "name=httpd state=present" -b -i inventory
```

---

## ⚙️ Service Management

Equivalent to `systemctl start --now httpd`.

```yaml
---
- name: Start httpd service
  hosts: all
  become: true

  tasks:
    - name: Ensure httpd is running and enabled
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

| Parameter | Values |
|-----------|--------|
| `state` | `started`, `stopped`, `restarted`, `reloaded` |
| `enabled` | `true` (boot), `false` (no boot) |

---

## 📁 File Module

Create directories and files with ownership and permissions.

```yaml
---
- hosts: all
  become: true

  tasks:
    - name: Create directory
      ansible.builtin.file:
        path: /opt/app
        state: directory
        owner: app
        group: app
        mode: "0755"

    - name: Create file
      ansible.builtin.file:
        path: /opt/app/index.html
        state: touch
        owner: app
        group: app
        mode: "0644"
```

### Common `state` Values

| State | Action |
|-------|--------|
| `directory` | Create a directory |
| `file` | Create an empty file |
| `touch` | Create file if missing, update timestamp |
| `absent` | Delete file or directory |
| `link` | Create a symlink |

---

## 🔍 Ansible Facts

Facts are variables automatically gathered from each host (OS, IP, memory, disks, etc.). Use them for conditional tasks.

```bash
# Collect all facts for a host
ansible web1 -m setup -i inventory

# Filter specific facts
ansible web1 -m setup -a "filter=ansible_os_family" -i inventory
```

### Use Facts in a Playbook

```yaml
---
- hosts: myservers
  become: true

  tasks:
    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
      when: ansible_os_family == "RedHat"
```

> 💡 Facts run automatically at the start of each play. Disable with `gather_facts: false` if not needed.

---

## 🔥 Firewalld Module

Configure firewall rules on RHEL/CentOS systems.

```yaml
---
- hosts: myservers
  become: true

  tasks:
    - name: Configure firewall
      ansible.posix.firewalld:
        service: http
        port: 8080/tcp
        source: 192.0.0.0/24
        zone: public
        state: enabled
        permanent: true
        immediate: true
```

| Parameter | Description |
|-----------|-------------|
| `permanent` | Rule persists after reboot |
| `immediate` | Apply the rule right now |
| `service` | Open a predefined service (e.g. `http`, `https`) |
| `port` | Open a specific port (e.g. `8080/tcp`) |

---

## ⚙️ ansible.cfg

Set defaults so you don't need `-i inventory` on every command.

```ini
# ansible.cfg
[defaults]
inventory = ./inventory
host_key_checking = False
retry_files_enabled = False
```

```bash
# Place ansible.cfg in the project root, then run:
ansible myservers -m ping
ansible-playbook site.yml
```

---

## 📝 Full Sample Playbook

```yaml
---
- hosts: myservers
  become: true

  tasks:
    - name: Create training directory
      ansible.builtin.file:
        path: /opt/training
        state: directory
        owner: training
        group: training
        mode: "0755"

    - name: Create index file
      ansible.builtin.file:
        path: /opt/training/index.html
        state: touch
        owner: training
        group: training
        mode: "0644"
```

---

## 🛠️ Useful Ad-Hoc Module Commands

```bash
# Copy a file
ansible all -m copy -a "src=./app.conf dest=/etc/app/app.conf" -b -i inventory

# Run a script
ansible web -m script -a "./deploy.sh" -b -i inventory

# Find files
ansible all -m find -a "paths=/var/log age=7d" -b -i inventory

# Template a Jinja2 file
ansible web -m template -a "src=nginx.conf.j2 dest=/etc/nginx/nginx.conf" -b -i inventory
```
