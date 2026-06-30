# 🔢 Ansible Variables

Variables store information that can differ per host, group, or play. Use them to keep playbooks reusable and DRY.

---

## 📐 Define Variables in a Playbook

```yaml
---
- name: Variable playbook test
  hosts: localhost

  vars:
    var_one: awesome
    var_two: ansible is
    var_three: "{{ var_two }} {{ var_one }}"

  tasks:
    - name: Print var_three
      ansible.builtin.debug:
        msg: "{{ var_three }}"
```

**Output:**

```
"msg": "ansible is awesome"
```

> 💡 Use `{{ }}` to reference variables. Jinja2 expressions work inside double curly braces.

---

## 🌐 Use Variables in Tasks

```yaml
---
- name: Add DNS server to resolv.conf
  hosts: localhost
  become: true

  vars:
    dns_server: 10.1.250.10

  tasks:
    - name: Set nameserver
      ansible.builtin.lineinfile:
        path: /etc/resolv.conf
        line: "nameserver {{ dns_server }}"
        create: true
```

---

## 📁 External Variable Files

Store variables in separate files and load them with `vars_files`.

### `group_vars/web.yml`

```yaml
http_port: 8081
snmp_port: "161-162"
```

### Playbook Using External Vars

```yaml
---
- name: Set firewall configurations
  hosts: web
  become: true

  vars_files:
    - group_vars/web.yml

  tasks:
    - name: Enable HTTPS service
      ansible.posix.firewalld:
        service: https
        permanent: true
        state: enabled

    - name: Open HTTP port
      ansible.posix.firewalld:
        port: "{{ http_port }}/tcp"
        permanent: true
        state: enabled

    - name: Open SNMP port
      ansible.posix.firewalld:
        port: "{{ snmp_port }}/udp"
        permanent: true
        state: enabled
```

---

## 🏷️ Variable Types

### String

```yaml
username: "admin"
```

### Number

```yaml
max_connections: 100
```

### Boolean

```yaml
debug_mode: true
```

### List

```yaml
packages:
  - nginx
  - postgresql
  - git
```

### Dictionary

```yaml
user:
  name: admin
  password: secret
```

### Access List/Dictionary Items

```yaml
# First package
{{ packages[0] }}        # nginx

# Dictionary key
{{ user.name }}          # admin
```

---

## ✨ Magic Variables

Ansible provides built-in variables you can use without defining them.

| Variable | Description |
|----------|-------------|
| `inventory_hostname` | Host name as defined in inventory |
| `ansible_host` | IP or hostname used for SSH connection |
| `hostvars` | Dictionary of all variables for all hosts |
| `groups` | List of groups the current host belongs to |
| `group_names` | Groups the current host is a member of |
| `ansible_os_family` | OS family (`RedHat`, `Debian`, etc.) |
| `ansible_distribution` | OS name (`CentOS`, `Ubuntu`, etc.) |

---

## 🔗 hostvars — Access Another Host's Variables

### Inventory

```ini
[myservers]
web1
web2
```

### `host_vars/web1.yml`

```yaml
ansible_host: 172.20.1.100
```

### `host_vars/web2.yml`

```yaml
ansible_host: 172.20.1.101
dns_server: 10.5.5.4
```

### Print Another Host's Variable

```yaml
---
- name: Print DNS server from web2
  hosts: all

  tasks:
    - name: Show web2 DNS server
      ansible.builtin.debug:
        msg: "{{ hostvars['web2'].dns_server }}"
```

**Output on any host:**

```
"msg": "10.5.5.4"
```

---

## 📍 Variable Precedence (High → Low)

Ansible resolves variables in this order (highest wins):

1. Extra vars (`-e` on CLI)
2. Task vars
3. Block vars
4. Role / include vars
5. Play vars
6. `host_vars`
7. `group_vars`
8. Inventory vars
9. Facts
10. Role defaults

```bash
# Override at runtime
ansible-playbook site.yml -e "http_port=9090"
```

---

## 🛠️ Quick Reference

```yaml
# Inline in play
vars:
  app_port: 8080

# From file
vars_files:
  - vars/common.yml

# Register command output
- command: cat /etc/hostname
  register: hostname_result

- debug:
    msg: "{{ hostname_result.stdout }}"

# Set a fact manually
- set_fact:
    env_name: production

# Conditional with variable
- debug:
    msg: "Production server"
  when: env_name == "production"
```
