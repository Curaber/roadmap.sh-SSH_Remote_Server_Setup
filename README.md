# roadmap.sh-SSH_Remote_Server_Setup
Setting up a basic remote linux server and configuring it to allow SSH. 

Project from: https://roadmap.sh/projects/ssh-remote-server-setup

## Goals
- Set up a Ubuntu server VM. 
- Create two new SSH key pairs and add them to your server. 
- Set up the server to only use public key authentication. 
- Connect to the server using an alias. 
- Install and configure Fail2Ban. 

## Steps
### 1. Set up a Ubuntu server VM
- Downloaded a Ubuntu 25.04 ISO image
- Enabled Hyper V in Windows
- Created and configured a VM inside Hyper V
- Installed Ubuntu onto the VM

### 2. Create two new SSH key pairs and add them to your server. 
- Created two key pairs on the local machine using:
```sh
ssh-keygen -t rsa -f kljuc
ssh-keygen -t rsa -f kljuc2
```
- Copied the public keys to the `~/.ssh/authorized_keys` file inside the VM (since Windows did not have `ssh-copy-id` this was done manually)
- Tested by connecting to the server using: 
```sh
ssh <user>@<UbuntuServerIP>
```

### 3. Set up the server to only use public key authentication. 
- Disabled password login by creating `00-only-pubkey-auth.conf` inside `/etc/ssh/sshd_config.d/` containing: 
```
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePam no
```
- Tested if the password and CRA logins are disabled by running:
```sh
ssh -v -n \
  -o Batchmode=yes \
  -o StrictHostKeyChecking=no \
  -o UserKnownHostsFile=/dev/null \
  DOES_NOT_EXIST@localhost
```
and looking for: 
```sh
debug1: Authentications that can continue: publickey
```

### 4. Connect to the server using an alias. 
- To allow connecting to the remote machine using an alias, created a `config` file inside `C:\Users\<user>\.ssh` and added: 
```
Host ubuntuserver
    HostName <UbuntuServerIP>
    User <user>
    IdentityFile "C:\Users\<user>\kljuc"

Host ubuntuserver2
    HostName <UbuntuServerIP>
    User <user>
    IdentityFile "C:\Users\<user>\kljuc2"
```
- Tested by connecting to the server using: 
```sh
ssh ubuntuserver
ssh ubuntuserver2
```

### 5. Install and configure Fail2Ban. 
- Installed Fail2Ban using:
```sh
sudo apt update
sudo apt install fail2ban
```
- Configured by editing `/etc/fail2ban/jail.local`:
```
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
bantime = 600
findtime = 600
```
- Restarted Fail2Ban: 
```sh
sudo systemctl restart fail2ban
```
