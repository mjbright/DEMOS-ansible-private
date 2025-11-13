# Ansible Ad-Hoc Commands Demo

## Introduction

Ansible ad-hoc commands are one-line commands that allow you to execute simple tasks on remote hosts without writing a playbook. They are perfect for quick tasks, testing, and troubleshooting.

## Basic Syntax

```bash
ansible [host-pattern] -m [module] -a "[module options]"
```

- **host-pattern**: Target hosts or groups from inventory
- **-m**: Module to execute (default is `command`)
- **-a**: Module arguments

## Common Options

- `-i`: Specify inventory file
- `-u`: Specify remote user
- `-b` or `--become`: Run with privilege escalation (sudo)
- `-K` or `--ask-become-pass`: Prompt for privilege escalation password
- `-k` or `--ask-pass`: Prompt for connection password
- `-f`: Number of parallel processes (forks)
- `-v`, `-vv`, `-vvv`: Verbose output (increasing verbosity)
- `--private-key`: Specify SSH private key

## 1. Ping Module - Testing Connectivity

### Basic ping to all hosts
```bash
ansible all -m ping
```

### Ping specific host
```bash
ansible webserver1 -m ping
```

### Ping a group of hosts
```bash
ansible webservers -m ping
```

### Ping with custom inventory
```bash
ansible all -i inventory.ini -m ping
```

## 2. Command Module - Running Simple Commands

### Check disk space on all hosts
```bash
ansible all -m command -a "df -h"
```

### Check uptime
```bash
ansible all -m command -a "uptime"
```

### List files in a directory
```bash
ansible all -m command -a "ls -la /var/log"
```

### Get system information
```bash
ansible all -m command -a "uname -a"
```

**Note**: The `command` module doesn't support shell variables, pipes, or redirects. For those, use the `shell` module.

## 3. Shell Module - Running Shell Commands

### Using pipes and redirects
```bash
ansible all -m shell -a "ps aux | grep nginx"
```

### Using environment variables
```bash
ansible all -m shell -a "echo $PATH"
```

### Command with output redirection
```bash
ansible all -m shell -a "cat /etc/hosts > /tmp/hosts.backup"
```

### Complex shell command
```bash
ansible all -m shell -a "if [ -f /etc/redhat-release ]; then cat /etc/redhat-release; fi"
```

## 4. Copy Module - Copying Files

### Copy file to remote hosts
```bash
ansible all -m copy -a "src=/local/path/file.txt dest=/remote/path/file.txt"
```

### Copy with specific permissions
```bash
ansible all -m copy -a "src=config.conf dest=/etc/app/config.conf mode=0644 owner=root group=root"
```

### Copy with backup
```bash
ansible all -m copy -a "src=app.conf dest=/etc/app.conf backup=yes"
```

### Copy with content inline
```bash
ansible all -m copy -a "content='Hello World\n' dest=/tmp/hello.txt"
```

## 5. File Module - Managing Files and Directories

### Create a directory
```bash
ansible all -m file -a "path=/opt/myapp state=directory mode=0755"
```

### Create a file
```bash
ansible all -m file -a "path=/tmp/testfile.txt state=touch mode=0644"
```

### Delete a file
```bash
ansible all -m file -a "path=/tmp/oldfile.txt state=absent"
```

### Create a symbolic link
```bash
ansible all -m file -a "src=/usr/bin/python3 dest=/usr/bin/python state=link"
```

### Change file permissions
```bash
ansible all -m file -a "path=/var/www/html mode=0755 owner=www-data group=www-data recurse=yes"
```

## 6. Package Management

### Using apt (Debian/Ubuntu)
```bash
# Install a package
ansible debian-servers -m apt -a "name=nginx state=present" -b

# Update package cache
ansible debian-servers -m apt -a "update_cache=yes" -b

# Install specific version
ansible debian-servers -m apt -a "name=nginx=1.18.0-0ubuntu1 state=present" -b

# Remove a package
ansible debian-servers -m apt -a "name=nginx state=absent" -b

# Upgrade all packages
ansible debian-servers -m apt -a "upgrade=dist" -b
```

### Using yum/dnf (RedHat/CentOS)
```bash
# Install a package
ansible redhat-servers -m yum -a "name=httpd state=present" -b

# Install multiple packages
ansible redhat-servers -m yum -a "name=httpd,vim,git state=present" -b

# Remove a package
ansible redhat-servers -m yum -a "name=httpd state=absent" -b

# Update all packages
ansible redhat-servers -m yum -a "name=* state=latest" -b
```

### Using package module (generic)
```bash
# Works across different package managers
ansible all -m package -a "name=git state=present" -b
```

## 7. Service Management

### Start a service
```bash
ansible all -m service -a "name=nginx state=started" -b
```

### Stop a service
```bash
ansible all -m service -a "name=nginx state=stopped" -b
```

### Restart a service
```bash
ansible all -m service -a "name=nginx state=restarted" -b
```

### Enable service on boot
```bash
ansible all -m service -a "name=nginx enabled=yes" -b
```

### Check service status
```bash
ansible all -m service -a "name=nginx state=started enabled=yes" -b
```

### Reload service configuration
```bash
ansible all -m systemd -a "name=nginx state=reloaded" -b
```

## 8. User Management

### Create a user
```bash
ansible all -m user -a "name=john state=present" -b
```

### Create user with specific UID and shell
```bash
ansible all -m user -a "name=john uid=1500 shell=/bin/bash state=present" -b
```

### Add user to groups
```bash
ansible all -m user -a "name=john groups=sudo,docker append=yes" -b
```

### Remove a user
```bash
ansible all -m user -a "name=john state=absent remove=yes" -b
```

### Create user with password
```bash
ansible all -m user -a "name=john password={{ 'mypassword' | password_hash('sha512') }} state=present" -b
```

## 9. Group Management

### Create a group
```bash
ansible all -m group -a "name=developers state=present" -b
```

### Create group with specific GID
```bash
ansible all -m group -a "name=developers gid=3000 state=present" -b
```

### Remove a group
```bash
ansible all -m group -a "name=developers state=absent" -b
```

## 10. Gathering Facts

### Gather all facts
```bash
ansible all -m setup
```

### Filter facts by pattern
```bash
ansible all -m setup -a "filter=ansible_eth*"
```

### Get only network facts
```bash
ansible all -m setup -a "filter=ansible_default_ipv4"
```

### Get memory information
```bash
ansible all -m setup -a "filter=ansible_memory_mb"
```

### Save facts to file
```bash
ansible all -m setup --tree /tmp/facts
```

## 11. Git Module - Repository Management

### Clone a repository
```bash
ansible all -m git -a "repo=https://github.com/user/repo.git dest=/opt/myapp"
```

### Clone specific branch
```bash
ansible all -m git -a "repo=https://github.com/user/repo.git dest=/opt/myapp version=develop"
```

### Update repository
```bash
ansible all -m git -a "repo=https://github.com/user/repo.git dest=/opt/myapp update=yes"
```

## 12. Archive and Unarchive

### Create an archive
```bash
ansible all -m archive -a "path=/var/log/myapp dest=/tmp/logs.tar.gz format=gz"
```

### Extract an archive
```bash
ansible all -m unarchive -a "src=/tmp/archive.tar.gz dest=/opt/app remote_src=yes"
```

### Extract from local file to remote
```bash
ansible all -m unarchive -a "src=/local/archive.tar.gz dest=/opt/app"
```

## 13. Cron Management

### Add a cron job
```bash
ansible all -m cron -a "name='daily backup' minute=0 hour=2 job='/usr/local/bin/backup.sh'"
```

### Remove a cron job
```bash
ansible all -m cron -a "name='daily backup' state=absent"
```

### Disable a cron job (comment it out)
```bash
ansible all -m cron -a "name='daily backup' disabled=yes"
```

## 14. Mount Management

### Mount a filesystem
```bash
ansible all -m mount -a "path=/mnt/data src=/dev/sdb1 fstype=ext4 state=mounted" -b
```

### Add to fstab without mounting
```bash
ansible all -m mount -a "path=/mnt/data src=/dev/sdb1 fstype=ext4 state=present" -b
```

### Unmount and remove from fstab
```bash
ansible all -m mount -a "path=/mnt/data state=absent" -b
```

## 15. Reboot and Wait

### Reboot hosts
```bash
ansible all -m reboot -b
```

### Reboot with custom timeout
```bash
ansible all -m reboot -a "reboot_timeout=600" -b
```

### Wait for hosts to come back online
```bash
ansible all -m wait_for_connection -a "delay=10 timeout=300"
```

## 16. Debug and Testing

### Display a message
```bash
ansible localhost -m debug -a "msg='Hello from Ansible'"
```

### Display variable value
```bash
ansible all -m debug -a "var=ansible_distribution"
```

## 17. Working with Multiple Hosts

### Run on specific hosts using limit
```bash
ansible all -m ping --limit "webserver1,webserver2"
```

### Exclude hosts
```bash
ansible all -m ping --limit "all:!webserver1"
```

### Run on hosts matching pattern
```bash
ansible "web*" -m ping
```

### Run with parallel execution
```bash
ansible all -m ping -f 10
```

## 18. Fetch Module - Copy Files from Remote to Local

### Fetch file from remote hosts
```bash
ansible all -m fetch -a "src=/etc/hostname dest=/tmp/hostnames/"
```

### Fetch with flat directory structure
```bash
ansible all -m fetch -a "src=/etc/hosts dest=/tmp/hosts/ flat=yes"
```

## 19. Synchronize Module - Rsync Wrapper

### Sync directory to remote hosts
```bash
ansible all -m synchronize -a "src=/local/path/ dest=/remote/path/"
```

### Sync with delete option
```bash
ansible all -m synchronize -a "src=/local/path/ dest=/remote/path/ delete=yes"
```

## 20. Working with Variables

### Pass extra variables
```bash
ansible all -m debug -a "msg='Hello {{ username }}'" -e "username=John"
```

### Multiple variables
```bash
ansible all -m file -a "path={{ app_dir }}/config state=directory" -e "app_dir=/opt/myapp"
```

## Best Practices and Tips

1. **Always test with a single host first**
   ```bash
   ansible webserver1 -m ping
   ```

2. **Use check mode (dry-run) for testing**
   ```bash
   ansible all -m yum -a "name=nginx state=present" --check -b
   ```

3. **Use verbose mode for debugging**
   ```bash
   ansible all -m ping -vvv
   ```

4. **Limit concurrent connections for large inventories**
   ```bash
   ansible all -m ping -f 5
   ```

5. **Use ansible.cfg for common settings**
   - Create `ansible.cfg` in your working directory to set defaults

6. **Always use `become` for privileged operations**
   ```bash
   ansible all -m service -a "name=nginx state=started" -b
   ```

7. **Prefer specific modules over shell/command when available**
   - Use `apt` module instead of `shell -a "apt-get install"`
   - Use `file` module instead of `command -a "mkdir"`

8. **Test connectivity before running commands**
   ```bash
   ansible all -m ping && ansible all -m command -a "your-command"
   ```

## Common Use Cases

### Quick Security Audit
```bash
# Check for specific users
ansible all -m shell -a "grep '^user:' /etc/passwd"

# Check SSH configuration
ansible all -m command -a "cat /etc/ssh/sshd_config" -b

# Check open ports
ansible all -m shell -a "netstat -tlnp"
```

### System Maintenance
```bash
# Clear cache
ansible all -m shell -a "sync && echo 3 > /proc/sys/vm/drop_caches" -b

# Check disk usage
ansible all -m shell -a "df -h | grep -v tmpfs"

# Find large files
ansible all -m shell -a "find /var/log -type f -size +100M"
```

### Emergency Operations
```bash
# Kill a process by name
ansible all -m shell -a "pkill -9 stuck_process" -b

# Quick firewall rule
ansible all -m shell -a "iptables -A INPUT -p tcp --dport 8080 -j ACCEPT" -b

# Emergency log rotation
ansible all -m shell -a "truncate -s 0 /var/log/large.log" -b
```

## Conclusion

Ad-hoc commands are powerful for quick tasks, but for complex, repeatable operations, consider writing Ansible playbooks. They provide better structure, error handling, and reusability.

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
