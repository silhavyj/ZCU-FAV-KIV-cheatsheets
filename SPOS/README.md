
# General tools & Commands
```bash
apt-get update
apt-get install vim mc net-tools dnsutils curl netcat
```
``` bash
echo ":set mouse-=a
:set tabstop=4
syntax on" > ~/.vimrc

echo SELECTED_EDITOR="/usr/bin/vim.tiny" > ~/.selected_editor
```
```bash
netstat -tupln | grep 22
grep -i token -R . 
find /etc/bind/ -name "*.local"
```
```bash
rsync -rva /var/lib/postgresql/* /mnt/databases/postgres/
```
```bash
timedatectl
dpkg-reconfigure tzdata
```
```bash
usermod -aG test-group pepa
gpasswd -d pepa test-group
```
```bash
useradd -r -m -s /usr/sbin/nologin mail001
useradd -r -m -s /usr/sbin/nologin mail002
echo "Test-1" | mail -s "Test mail" mail001@mail.silhavyj.spos-test.spos
adduser kontrola
```
```
https://crontab.guru/
```
```
ip a a 10.228.67.42/24 dev ens18
```
