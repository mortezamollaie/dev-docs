# 📘 Ansible Basics

Ansible is an **automation** and **configuration management** tool that connects to servers over SSH (Linux) and WinRM (Windows).

---

## 🖥️ Nodes

### Control Node
The machine where Ansible is installed and from which you run playbooks and commands.

### Managed Node
The servers that are managed and configured by the control node.

> 🔗 Ansible connects to managed nodes via **SSH** (Linux) or **WinRM** (Windows).

---

## 📄 Data Formats

### XML — Key/Value Structure

```xml
<name>morteza</name>
```

### JSON — Key/Value Structure

```json
{
  "name": "morteza"
}
```

### YAML — Ansible's Default Format

```yaml
name: food1
price: 300
```

---

## 📝 YAML Structure

### Dictionary (Key-Value)

```yaml
menu:
  name: food1
  price: 300
```

### List

```yaml
menu:
  - food1
  - food2
  - food3
```

### ⚠️ Important Note
YAML is **indentation-sensitive**. Incorrect spacing will break dictionary or list structure.

### List + Dictionary Combo (Common in Ansible)

```yaml
menu:
  - name: food1
    price: 300
  - name: food2
    price: 400
```

---

## 📋 Inventory

**Inventory** is the file that defines servers and their groups for Ansible. Ansible works across multiple systems at once, and inventory is usually stored as a file.

- You can define multiple **groups**
- You can set variables per group or per host

### Sample Inventory (INI Format)

```ini
[mail]
server1.company.com
server2.company.com

[db]
server3.company.com
server4.company.com

[web]
server5.company.com
server6.company.com

10.42.0.[100:130]
10.42.0.2
host1.morteza.mollaie
```

> 💡 `10.42.0.[100:130]` is a **range** — equivalent to `10.42.0.100` through `10.42.0.130`.

### Inventory Variables

```ini
[myservers]
myserver01 ansible_host=10.42.0.2

[web]
web1 ansible_host=172.20.1.100 ansible_ssh_pass=Passw0rd
```

### Inventory Formats

| Format | Description |
|--------|-------------|
| **INI** | Classic, simple format (default) |
| **YAML** | More structured format |

#### Sample Inventory (YAML Format)

```yaml
all:
  children:
    web:
      hosts:
        web1:
          ansible_host: 172.20.1.100
        web2:
          ansible_host: 172.20.1.101
    db:
      hosts:
        db1:
          ansible_host: 172.20.1.200
```

---

## 🏢 Hosts & Groups

In large organizations, servers are grouped by **role**:

- Less repetition in configuration
- Easier management
- Run playbooks only on relevant servers

### Sample Grouping

```ini
[web_servers]
web1
web2
web3
```

---

## 📁 Host Variables & Group Variables

Instead of repeating variables in the inventory file, store them in separate files.

### Folder Structure

```
inventory/
├── hosts
├── host_vars/
│   ├── web1.yml
│   ├── web2.yml
│   └── web3.yml
└── group_vars/
    └── web_servers.yml
```

### `host_vars/web1.yml`

```yaml
ansible_host: 172.20.1.100
dns_server: 10.1.1.5
```

### `host_vars/web2.yml`

```yaml
ansible_host: 172.20.1.101
dns_server: 10.1.1.5
```

### `group_vars/web_servers.yml` (Shared Group Variable)

```yaml
dns_server: 10.1.1.5
```

> 💡 Put shared variables in `group_vars` and host-specific variables in `host_vars`.

---

## 🔐 Remote Connection Methods

### 1. Username & Password

```yaml
# host_vars/web1.yml
ansible_host: 172.20.1.100
ansible_user: root
ansible_ssh_pass: Miripass
```

### 2. SSH Key (More Secure)

```yaml
# host_vars/web2.yml
ansible_host: 172.20.1.101
ansible_user: root
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

### SSH Key Setup Commands

```bash
# Generate SSH key pair
ssh-keygen
# Output: id_rsa (private) and id_rsa.pub (public)

# Copy public key to the server
ssh-copy-id root@server1
# Output: Number of key(s) added: 1

# Verify key on the server
cat ~/.ssh/authorized_keys
# Output: ssh-rsa AAAAB3NzaC1yc2E...

# Test connection with the key
ssh -i id_rsa root@server1
# Output: Successfully Logged In!
```

---

## ⚙️ Install Ansible on the Control Node

| OS | Install Command |
|----|-----------------|
| RedHat | `sudo yum install ansible` |
| Fedora | `sudo dnf install ansible` |
| Ubuntu | `sudo apt-get install ansible` |
| PIP | `sudo pip install ansible` |

### Other Options
- Install from Git source
- Build an RPM manually

### Sample Install on CentOS/RHEL

```bash
# Detect OS type
cat /etc/os-release

# Install EPEL (for RHEL/CentOS)
sudo yum install epel-release

# Install Ansible
sudo yum install ansible
```

---

## 🚀 Initial Setup

### Default `/etc/ansible` Structure

```bash
cd /etc/ansible
# Files: ansible.cfg | hosts (inventory) | roles/
```

### Create Inventory & Host Variables

```bash
# Create inventory file (both "hosts" and "inventory" names are valid)
touch inventory
```

```ini
# inventory
[myservers]
server1

[db]
194.5.192.37
```

```bash
mkdir host_vars
touch host_vars/server1.yml
```

```yaml
# host_vars/server1.yml
---
ansible_host: 130.185.121.156
ansible_user: root
ansible_ssh_pass: Mortezapass
```

### Test Connection

```bash
ansible myservers -m ping -i inventory
```

Successful output:

```
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> 🔑 Recommendation: After the initial test, remove `ansible_ssh_pass` and migrate to **SSH keys**.
