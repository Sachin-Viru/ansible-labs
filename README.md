# 🧪 Ansible Labs Project

Welcome to the **Ansible Labs** repository! This project helps you learn and test Ansible automation using local Vagrant-based virtual machines.

---

## 📁 Project Structure

```
ansible-labs/
├── docs/                          # Documentation & screenshots
│   ├── ansible_commands.md        # Cheatsheet of useful Ansible commands
│   ├── vagrant_setup.md           # Guide to set up Vagrant lab
│   └── images/                    # Images used in documentation
│       ├── authorized_keys.png
│       ├── sshd_config.png
│       ├── ssh-restart.png
│       ├── ssh-root-login.png
│       ├── sudo-su-root.png
│       ├── vagrant-ssh-host0.png
│       ├── vagrant-up-output.png
│       └── nginx-playbook-output.png
│
├── inventory/                     # Inventory file for Ansible
│   └── hosts.ini
│
├── playbooks/                     # Ansible playbooks
│   └── nginx.yml                  # Example playbook to configure Nginx
│
├── roles/                         # Reusable Ansible roles
│   └── apache/                    # Apache role
│       ├── tasks/                 # Task definitions
│       │   └── main.yml
│       ├── templates/             # Jinja2 templates (currently empty)
│       └── vars/                  # Variables for the role
│           └── main.yml
│
├── vagrant/                       # Vagrant configuration files
│   └── Vagrantfile
│
├── README.md                      # Project overview
```

---

## 🚀 Getting Started

### 📦 Requirements

- [x] **Linux OS** (tested on Ubuntu/Debian)
- [x] `vagrant`
- [x] `virtualbox`
- [x] `ansible`

---

### ⚙️ Install Vagrant & VirtualBox

```bash
sudo apt update
sudo apt install vagrant
```

> Download and install VirtualBox from:  
> 🔗 https://www.virtualbox.org/wiki/Downloads

### OR

```bash
sudo apt install virtualbox
```

### ⚙️ Install Ansible

```bash
sudo apt install ansible
```

### ✔️ Check Version

```bash
ansible --version
```

---

### 🔐 BIOS Setup (Important)

✅ **Enable**: Virtualization (VT-x / AMD-V)  
🚫 **Disable**: Secure Boot (if VM fails to boot)

---

## 🖥️ Launch Vagrant Hosts

```bash
cd ansible-labs/
sudo vagrant up
```

- This will spin up `host0`, `host1`, and `host2` as defined in the `Vagrantfile`.
- SSH into a host using:

```bash
sudo vagrant ssh host0
sudo su   # To switch to root
```

---

## 📚 Documentation

- 📘 [Vagrant Setup Guide](docs/vagrant_setup.md)  
  Covers BIOS settings, SSH root login, key management, etc.

- 📘 [Ansible Commands Cheatsheet](docs/ansible_commands.md)  
  Quick reference for useful ad-hoc commands and playbook runs

---

## ▶️ Running Ansible Playbooks

### 🔁 Check VM connectivity

```bash
ansible all -i inventory/hosts.ini -m ping
```

### 🛠️ Run a playbook

```bash
ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml
```

---

## 🌐 Nginx Setup Using Ansible

The `nginx.yml` playbook in `playbooks/` is designed to:

- Install **Nginx** on `host2`
- Start and enable the Nginx service
- Create a custom directory `/var/www/html/sachin`
- Upload a custom `sachin.html` page to that directory
- Copy and modify the default Nginx config to host the custom page
- Enable the new config and restart Nginx

This demonstrates how to automate end-to-end web server setup using Ansible.

📸 **Playbook Output Screenshot**  
> ![nginx-playbook-output](docs/images/nginx-playbook-output.png)

---

### 📜 `nginx.yml` Playbook

```yaml
---
- name: install another webserver nginx and hosts the website in host2 server 
  hosts: all
  become: true
  become_user: root
  remote_user: root
  gather_facts: false
  tasks:
    - name: install the nginx-web server in host2
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: start the nginx-web server in host2
      service:
        name: nginx
        state: started

    - name: enable the nginx-web server in host2
      service:
        name: nginx
        enabled: true

    - name: create custom directory in /var/www/html/ in host2
      file:
        path: /var/www/html/sachin
        state: directory
        owner: www-data
        group: www-data
        mode: 0755

    - name: create custom html file in /var/www/html/sachin in host2
      file: 
        path: /var/www/html/sachin/sachin.html
        state: touch
        owner: www-data
        group: www-data
        mode: 0644

    - name: copying custom html file from local to host2 
      copy:
        src: /home/sachin/Documents/sachin.html
        dest: /var/www/html/sachin/sachin.html
        owner: www-data
        group: www-data
        mode: 0644

    - name: copying 000-default.conf to /etc/nginx/sites-available/sachin.conf in host2
      copy:
        src: /etc/nginx/sites-available/default  
        dest: /etc/nginx/sites-available/sachin.conf
        owner: root
        group: root
        mode: 0644
        remote_src: yes

    - name: remove the default_hosts in sachin.conf file
      replace:
        path: /etc/nginx/sites-available/sachin.conf
        regexp: '^(\s*)listen 80 default_server;\s+.+'     
        replace: '        listen 80;'

    - name: remove the listen [::]:80 default_server;
      replace:
        path: /etc/nginx/sites-available/sachin.conf
        regexp: '^(\s*)listen [::]:80 default_server;\s+.+'
        replace: '        listen [::]:80;'  

    - name: update server_name to IP
      replace:
        path: /etc/nginx/sites-available/sachin.conf
        regexp: '^(\s*)server_name\s+.+'
        replace: '        server_name 192.168.56.10;'

    - name: update root directory
      replace:
        path: /etc/nginx/sites-available/sachin.conf
        regexp: '^(\s*)root\s+.+'
        replace: '        root /var/www/html/sachin;'

    - name: link config to sites-enabled
      command: ln -s /etc/nginx/sites-available/sachin.conf /etc/nginx/sites-enabled 
      register: ln_to_output
      ignore_errors: true

    - name: check output of ln command
      debug:
        msg: "{{ ln_to_output.stdout }}"   

    - name: check error of ln command
      debug: 
        msg: "{{ ln_to_output.stderr }}"    

    - name: check nginx config test 
      command: nginx -t
      register: nginx_test_output
      ignore_errors: true

    - name: check output of nginx test
      debug:
        msg: "{{ nginx_test_output.stdout }}"   

    - name: check error of nginx test
      debug:
        msg: "{{ nginx_test_output.stderr }}" 

    - name: restart nginx after config change
      service:
        name: nginx
        state: restarted

    - name: check the status of nginx
      command: systemctl status nginx
      register: nginx_status   

    - name: show nginx status
      debug:
        msg: "{{ nginx_status.stdout }}"   
```

---

## 📸 Screenshot Previews

You’ll find helpful screenshots in the `docs/images/` folder.

> Example:
> ![vagrant up](docs/images/vagrant-up-output.png)

---

## 🔧 Useful Vagrant Commands

| Command                    | Description                          |
|---------------------------|--------------------------------------|
| `vagrant up`              | Start all VMs                        |
| `vagrant ssh host0`       | SSH into a specific VM               |
| `vagrant halt`            | Shut down VMs                        |
| `vagrant destroy`         | Remove all VMs                       |
| `vagrant status`          | Show VM status                       |
| `vagrant reload`          | Restart and reload config            |

---

## 📌 Notes

- Always login as `vagrant` and `sudo su` to root.
- Don’t replace Vagrant's SSH keys unless necessary.
- Configure root login only from inside the VM.

---

## 🧠 Author

Built by 🚀 *Sachin-viru!*  
Feel free to fork and extend this lab for your own Ansible experiments.

---

## 📜 License

MIT License – Use freely, learn aggressively.

