# 🚀 Ansible Commands Cheatsheet

This file contains the most useful **Ansible commands** for quick reference.  
👉 Default inventory file used: `inventory/hosts.ini`

---

## 🗂️ Inventory Commands

🔹 Check if hosts are reachable:
```bash
ansible all -i inventory/hosts.ini -m ping

🔹 List all hosts from inventory:
```bash
ansible all -i inventory/hosts.ini --list-hosts


## ⚡ Ad-hoc Commands

🖥️ Run uptime on all hosts:
```bash
ansible all -i inventory/hosts.ini -m shell -a "uptime"

💾 Check disk usage:
```bash
ansible all -i inventory/hosts.ini -m shell -a "df -h"
🧠 Check memory:
ansible all -i inventory/hosts.ini -m shell -a "free -m"
📂 Copy a file to remote hosts:
ansible all -i inventory/hosts.ini -m copy -a "src=/etc/hosts dest=/tmp/hosts"
📦 Install a package (example: nginx):
ansible all -i inventory/hosts.ini -b -m service -a "name=nginx state=restarted"

## 📜 Playbook Commands
▶️ Run a playbook:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml
🔍 Run with verbose output:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml -v
⚙️ Run with extra variables:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml -e "myvar=value"
✅ Check playbook syntax:
ansible-playbook playbooks/apache.yml --syntax-check
🧪 Dry-run (check what would change):
ansible-playbook playbooks/apache.yml --check

## 🎯 Useful Options
🎯 Limit playbook to one host:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml --limit web1
🏷️ Run only specific tags:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml --tags "install"
🚫 Skip specific tags:
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml --skip-tags "config"

