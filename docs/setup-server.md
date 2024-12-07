# OS requirement
- Linux

# List command
```bash
# Create a new user and update its password
$ useradd -m <user_name>
$ passwd <user_name>
# Set default SHELL
$ chsh -s /bin/bash
# Change privilege
$ su root 
$ nano /etc/sudoers
# user_name ALL=(ALL)  ALL

# Change Hostname on Ubuntu via CLI (No Reboot)
$ sudo hostnamectl set-hostname new-hostname
```
