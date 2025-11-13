# Ansible Playbook Demo - Various Ways to Use Simple Playbooks

This document demonstrates various ways to use simple Ansible Playbooks with practical examples.

## Table of Contents
1. [Basic Playbook Structure](#basic-playbook-structure)
2. [Simple Ping Playbook](#simple-ping-playbook)
3. [Package Installation](#package-installation)
4. [File Operations](#file-operations)
5. [Using Variables](#using-variables)
6. [Using Handlers](#using-handlers)
7. [Using Tags](#using-tags)
8. [Using Conditionals](#using-conditionals)
9. [Using Loops](#using-loops)
10. [Using Roles](#using-roles)
11. [Multiple Plays in One Playbook](#multiple-plays-in-one-playbook)

---

## Basic Playbook Structure

Every Ansible playbook follows a basic YAML structure:

```yaml
---
- name: Playbook name
  hosts: target_hosts
  become: yes  # Optional: run as sudo
  tasks:
    - name: Task description
      module_name:
        parameter: value
```

---

## Simple Ping Playbook

The simplest playbook to verify connectivity to your hosts:

```yaml
---
- name: Ping all hosts
  hosts: all
  tasks:
    - name: Ping test
      ping:
```

**Run with:**
```bash
ansible-playbook ping.yml
```

---

## Package Installation

Install packages on target systems:

```yaml
---
- name: Install web server
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache on Debian/Ubuntu
      apt:
        name: apache2
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Install Apache on RedHat/CentOS
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"

    - name: Ensure Apache is running
      service:
        name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"
        state: started
        enabled: yes
```

**Run with:**
```bash
ansible-playbook install_webserver.yml
```

---

## File Operations

Manage files and directories:

```yaml
---
- name: File operations demo
  hosts: all
  become: yes
  tasks:
    - name: Create a directory
      file:
        path: /opt/myapp
        state: directory
        mode: '0755'
        owner: root
        group: root

    - name: Create a file with content
      copy:
        content: "Hello from Ansible!\n"
        dest: /opt/myapp/hello.txt
        mode: '0644'

    - name: Copy file from control node
      copy:
        src: /local/path/config.conf
        dest: /opt/myapp/config.conf
        backup: yes

    - name: Create symbolic link
      file:
        src: /opt/myapp/hello.txt
        dest: /tmp/hello_link.txt
        state: link

    - name: Remove a file
      file:
        path: /tmp/unwanted_file.txt
        state: absent
```

---

## Using Variables

Define and use variables in playbooks:

```yaml
---
- name: Variables demo
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
    app_name: "MyWebApp"
  tasks:
    - name: Display variables
      debug:
        msg: "Configuring {{ app_name }} on port {{ http_port }} with max {{ max_clients }} clients"

    - name: Create config from template
      template:
        src: templates/app.conf.j2
        dest: /etc/myapp/app.conf
      vars:
        custom_var: "custom_value"

    - name: Use variables in file content
      copy:
        content: |
          Application: {{ app_name }}
          Port: {{ http_port }}
          Max Clients: {{ max_clients }}
        dest: /opt/myapp/settings.txt
```

**External variables file (vars.yml):**
```yaml
---
http_port: 8080
max_clients: 500
app_name: "ProductionApp"
```

**Run with external vars:**
```bash
ansible-playbook playbook.yml -e @vars.yml
```

---

## Using Handlers

Handlers are tasks that run only when notified:

```yaml
---
- name: Handlers demo
  hosts: webservers
  become: yes
  tasks:
    - name: Copy Apache config
      copy:
        src: files/apache.conf
        dest: /etc/apache2/apache2.conf
      notify:
        - Restart Apache
        - Check Apache status

    - name: Update website content
      copy:
        content: "<h1>Welcome to {{ inventory_hostname }}</h1>"
        dest: /var/www/html/index.html
      notify: Reload Apache

  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted

    - name: Reload Apache
      service:
        name: apache2
        state: reloaded

    - name: Check Apache status
      command: systemctl status apache2
      register: apache_status
```

---

## Using Tags

Tags allow selective execution of tasks:

```yaml
---
- name: Tags demo
  hosts: all
  become: yes
  tasks:
    - name: Install packages
      apt:
        name:
          - vim
          - git
          - curl
        state: present
      tags:
        - packages
        - install

    - name: Configure system
      copy:
        content: "System configured by Ansible"
        dest: /etc/motd
      tags:
        - configuration
        - system

    - name: Setup users
      user:
        name: appuser
        state: present
        shell: /bin/bash
      tags:
        - users
        - security

    - name: Debug information
      debug:
        msg: "This always runs"
      tags:
        - always
```

**Run specific tags:**
```bash
# Run only tasks tagged with 'packages'
ansible-playbook playbook.yml --tags packages

# Run multiple tags
ansible-playbook playbook.yml --tags "packages,configuration"

# Skip specific tags
ansible-playbook playbook.yml --skip-tags users
```

---

## Using Conditionals

Execute tasks based on conditions:

```yaml
---
- name: Conditionals demo
  hosts: all
  become: yes
  tasks:
    - name: Install Apache on Ubuntu
      apt:
        name: apache2
        state: present
      when: ansible_distribution == "Ubuntu"

    - name: Install Apache on CentOS
      yum:
        name: httpd
        state: present
      when: ansible_distribution == "CentOS"

    - name: Only run on production servers
      debug:
        msg: "This is a production server"
      when: inventory_hostname in groups['production']

    - name: Check if file exists
      stat:
        path: /etc/myapp/config.conf
      register: config_file

    - name: Create config if it doesn't exist
      copy:
        content: "default config"
        dest: /etc/myapp/config.conf
      when: not config_file.stat.exists

    - name: Multiple conditions with AND
      service:
        name: apache2
        state: started
      when:
        - ansible_os_family == "Debian"
        - ansible_distribution_version >= "20.04"

    - name: Multiple conditions with OR
      debug:
        msg: "Running on Debian-based or RedHat-based system"
      when: ansible_os_family == "Debian" or ansible_os_family == "RedHat"
```

---

## Using Loops

Iterate over lists and dictionaries:

```yaml
---
- name: Loops demo
  hosts: all
  become: yes
  tasks:
    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - vim
        - git
        - htop
        - curl
        - wget

    - name: Create multiple users
      user:
        name: "{{ item }}"
        state: present
        shell: /bin/bash
      loop:
        - user1
        - user2
        - user3

    - name: Create users with different properties
      user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        shell: "{{ item.shell }}"
      loop:
        - { name: 'alice', uid: 1001, shell: '/bin/bash' }
        - { name: 'bob', uid: 1002, shell: '/bin/zsh' }
        - { name: 'charlie', uid: 1003, shell: '/bin/bash' }

    - name: Create multiple directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /opt/app1
        - /opt/app2
        - /opt/app3

    - name: Loop with dictionary
      debug:
        msg: "Server: {{ item.key }}, IP: {{ item.value }}"
      loop: "{{ servers | dict2items }}"
      vars:
        servers:
          web1: 192.168.1.10
          web2: 192.168.1.11
          db1: 192.168.1.20

    - name: Loop with index
      debug:
        msg: "Item {{ ansible_loop.index }}: {{ item }}"
      loop:
        - first
        - second
        - third
      loop_control:
        extended: yes
```

---

## Using Roles

Organize playbooks with roles:

**Directory structure:**
```
.
├── playbook.yml
└── roles/
    └── webserver/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── templates/
        │   └── index.html.j2
        ├── files/
        ├── vars/
        │   └── main.yml
        └── defaults/
            └── main.yml
```

**playbook.yml:**
```yaml
---
- name: Configure web servers using roles
  hosts: webservers
  become: yes
  roles:
    - webserver
    - { role: database, when: inventory_hostname in groups['dbservers'] }
```

**roles/webserver/tasks/main.yml:**
```yaml
---
- name: Install Apache
  apt:
    name: apache2
    state: present

- name: Copy website template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
  notify: Restart Apache

- name: Ensure Apache is running
  service:
    name: apache2
    state: started
    enabled: yes
```

**roles/webserver/handlers/main.yml:**
```yaml
---
- name: Restart Apache
  service:
    name: apache2
    state: restarted
```

**Run with:**
```bash
ansible-playbook playbook.yml
```

---

## Multiple Plays in One Playbook

Execute different plays for different host groups:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache
      service:
        name: apache2
        state: started

- name: Configure database servers
  hosts: dbservers
  become: yes
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present

    - name: Start MySQL
      service:
        name: mysql
        state: started

- name: Configure all servers
  hosts: all
  become: yes
  tasks:
    - name: Update all packages
      apt:
        upgrade: dist
        update_cache: yes

    - name: Install common tools
      apt:
        name:
          - vim
          - htop
          - net-tools
        state: present
```

---

## Additional Tips

### Check Mode (Dry Run)
```bash
ansible-playbook playbook.yml --check
```

### Verbose Output
```bash
ansible-playbook playbook.yml -v    # verbose
ansible-playbook playbook.yml -vv   # more verbose
ansible-playbook playbook.yml -vvv  # very verbose
```

### Limit to Specific Hosts
```bash
ansible-playbook playbook.yml --limit webserver1
ansible-playbook playbook.yml --limit "webservers:&production"
```

### List Tasks
```bash
ansible-playbook playbook.yml --list-tasks
```

### List Tags
```bash
ansible-playbook playbook.yml --list-tags
```

### Step-by-Step Execution
```bash
ansible-playbook playbook.yml --step
```

---

## Common Playbook Patterns

### Install, Configure, and Start Service
```yaml
---
- name: Complete service setup
  hosts: all
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload nginx

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

### Gather Facts and Use Them
```yaml
---
- name: Use system facts
  hosts: all
  tasks:
    - name: Display OS information
      debug:
        msg: "{{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: Display memory info
      debug:
        msg: "Total memory: {{ ansible_memtotal_mb }} MB"

    - name: Display network info
      debug:
        msg: "IP address: {{ ansible_default_ipv4.address }}"
```

---

This demo covers the most common patterns for writing simple Ansible playbooks. For more advanced usage, refer to the official Ansible documentation at https://docs.ansible.com/.
