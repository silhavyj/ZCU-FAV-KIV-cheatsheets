
# SSH
```bash
apt-get install openssh-server -y
```
#### generating keys and copying them onto the server
```bash
ssh-keygen -t rsa -b 4096 -C "kuba@spos"
ssh-copy-id root@192.168.1.3
ssh-copy-id -i ~/.ssh/mykey root@192.168.1.3
```
#### changing the config file on the server
```bash
vim /etc/ssh/sshd_config
```
```bash
PasswordAuthentication no
PermitRootLogin without-password 
PubkeyAuthentication yes
Port 22
AllowUsers root student
```
```bash
/etc/init.d/ssh restart
/etc/init.d/ssh status
```
#### remote logging to the server
```bash
ssh root@192.168.1.3
ssh -i ~/.ssh/mykey root@192.168.1.3
```
#### fail2ban
```bash
apt-get install fail2ban -y
```
```bash
vim /etc/fail2ban/jail.conf
```
```bash
[sshd]
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3
bantime = 7200
ignorip = 147.228.63.10
```
#### changing iptables
```bash
iptables -nvL
iptables -nvL --line-numbers

iptables -A INPUT -s 147.228.0.0/16 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

iptables -D INPUT -s 147.228.0.0/16 -p tcp --dport 22 -j ACCEPT
iptables -D INPUT 2

iptables-save > /etc/network/iptables
iptables-restore /etc/network/iptables
```
