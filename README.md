# Ansible Demos

A comprehensive collection of Ansible playbooks, roles, and examples for learning and demonstrating Ansible automation capabilities.

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Available Demos](#available-demos)
- [Usage Examples](#usage-examples)
- [Contributing](#contributing)

## 🎯 Overview

This repository contains practical Ansible examples and demonstrations covering:
- Basic playbook structures
- Inventory management
- Variable usage and precedence
- Role development
- File operations
- Package management
- Conditionals and loops

## 📁 Repository Structure

```
.
├── ansible.cfg              # Ansible configuration file
├── inventory/               # Inventory files
│   ├── hosts               # Static inventory example
│   └── localhost           # Localhost inventory for testing
├── playbooks/              # Example playbooks
│   ├── hello-world.yml     # Basic hello world example
│   ├── install-packages.yml # Package installation demo
│   ├── file-operations.yml  # File and directory operations
│   ├── variables-demo.yml   # Variables and conditionals
│   └── use-role.yml        # Example using roles
├── roles/                  # Ansible roles
│   └── webserver/          # Example webserver role
│       ├── tasks/          # Role tasks
│       ├── handlers/       # Event handlers
│       ├── templates/      # Jinja2 templates
│       ├── files/          # Static files
│       ├── vars/           # Role variables
│       ├── defaults/       # Default variables
│       └── meta/           # Role metadata
├── group_vars/             # Group-specific variables
│   ├── all.yml             # Variables for all hosts
│   ├── webservers.yml      # Variables for webservers
│   └── databases.yml       # Variables for databases
└── host_vars/              # Host-specific variables
    └── web1.yml            # Variables for web1 host
```

## 🔧 Prerequisites

- Ansible 2.9 or higher
- Python 3.x
- SSH access to target hosts (for remote execution)
- Sudo privileges (for some playbooks)

### Installing Ansible

**On Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ansible
```

**On macOS:**
```bash
brew install ansible
```

**Using pip:**
```bash
pip install ansible
```

## 🚀 Getting Started

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd DEMOS-ansible-private
   ```

2. **Verify Ansible installation:**
   ```bash
   ansible --version
   ```

3. **Test with the hello-world playbook:**
   ```bash
   ansible-playbook playbooks/hello-world.yml -i inventory/localhost
   ```

## 🎪 Available Demos

### 1. Hello World (`playbooks/hello-world.yml`)
The simplest Ansible playbook demonstrating:
- Basic playbook structure
- Debug module usage
- Accessing Ansible facts
- System information gathering

**Run it:**
```bash
ansible-playbook playbooks/hello-world.yml -i inventory/localhost
```

### 2. Package Installation (`playbooks/install-packages.yml`)
Demonstrates package management across different distributions:
- APT package management (Debian/Ubuntu)
- YUM package management (RedHat/CentOS)
- Conditional execution based on OS family
- Package verification

**Run it:**
```bash
ansible-playbook playbooks/install-packages.yml -i inventory/localhost
```

### 3. File Operations (`playbooks/file-operations.yml`)
Shows file and directory management:
- Creating directories
- Creating files with content
- Using loops
- Finding files
- Checking file status

**Run it:**
```bash
ansible-playbook playbooks/file-operations.yml -i inventory/localhost
```

### 4. Variables and Conditionals (`playbooks/variables-demo.yml`)
Explores variable usage and conditional logic:
- Variable definition and usage
- Conditional task execution
- Fact-based conditionals
- Setting dynamic facts
- Using filters and loops

**Run it:**
```bash
ansible-playbook playbooks/variables-demo.yml -i inventory/localhost
```

### 5. Using Roles (`playbooks/use-role.yml`)
Demonstrates role-based organization:
- Role structure and best practices
- Template usage
- Handler implementation
- Variable precedence
- Service management

**Run it:**
```bash
ansible-playbook playbooks/use-role.yml -i inventory/localhost
```

## 📚 Usage Examples

### Running Playbooks with Different Inventories

```bash
# Use localhost inventory
ansible-playbook playbooks/hello-world.yml -i inventory/localhost

# Use static inventory
ansible-playbook playbooks/install-packages.yml -i inventory/hosts

# Limit to specific hosts
ansible-playbook playbooks/hello-world.yml -i inventory/hosts --limit webservers

# Use tags (if defined in playbook)
ansible-playbook playbooks/install-packages.yml -i inventory/localhost --tags packages
```

### Checking Playbook Syntax

```bash
ansible-playbook playbooks/hello-world.yml --syntax-check
```

### Dry Run (Check Mode)

```bash
ansible-playbook playbooks/file-operations.yml -i inventory/localhost --check
```

### Verbose Output

```bash
ansible-playbook playbooks/hello-world.yml -i inventory/localhost -v
# More verbose: -vv, -vvv, -vvvv
```

### Using Vault for Sensitive Data

```bash
# Create encrypted file
ansible-vault create secret.yml

# Edit encrypted file
ansible-vault edit secret.yml

# Run playbook with vault
ansible-playbook playbooks/hello-world.yml -i inventory/localhost --ask-vault-pass
```

## 🔍 Understanding Key Concepts

### Inventory
Inventory files define the hosts and groups that Ansible manages. Variables can be assigned at the host or group level.

### Playbooks
Playbooks are YAML files that define automation tasks. They contain plays, which map hosts to tasks.

### Roles
Roles provide a way to organize playbooks and reusable content. They follow a standardized directory structure.

### Variables
Variables can be defined at multiple levels with the following precedence (highest to lowest):
1. Extra vars (`-e` on command line)
2. Task vars
3. Block vars
4. Role and include vars
5. Set_facts
6. Registered vars
7. Play vars
8. Host vars
9. Group vars
10. Role defaults

### Handlers
Handlers are tasks that run only when notified by other tasks, typically used for service restarts.

## 🤝 Contributing

Feel free to add more examples and improve existing ones! Some ideas for additional demos:
- Docker container management
- Cloud provisioning (AWS, Azure, GCP)
- Kubernetes deployment
- Database setup and configuration
- Monitoring setup
- CI/CD integration

## 📝 Notes

- The `ansible.cfg` file in this repository sets default configurations
- Always test playbooks in a safe environment before production use
- Use `--check` mode to preview changes without making them
- The inventory files contain example IP addresses; update them for your environment

## 📖 Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Community roles
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

## 📄 License

This is a demo repository for educational purposes.

---

**Happy Automating! 🚀**
