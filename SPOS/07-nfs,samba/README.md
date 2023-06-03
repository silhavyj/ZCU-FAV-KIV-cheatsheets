
## NFS (Network File System)
```bash
apt-get install nfs-kernel-server
apt-get install nfs-common # for the client
```
#### Editing the exports file
```bash
vim /etc/exports
```
```bash
/srv/share1       10.228.67.0/24(rw) 147.228.67.0/24(ro)
/srv/share2       10.228.67.0/24(rw,no_root_squash)
/srv/share3       10.228.67.0/24(rw,no_subtree_check,all_squash,anonuid=6000,anongid=6000)
```
```
rw               - read/write
ro               - read only
no_root_squash   - remote root users are able to change any file on the shared file system
no_subtree_check - this option disables subtree checking, which has mild security implications, but can improve reliability in some circumstances
all_squash       - map all uids and gids to the anonymous user
sync             - all changes to the according filesystem are immediately flushed to disk
async            - 
```
#### Creating  the folders
```bash
mkdir /srv/share1
chmod 777 /srv/share1
```
#### Creating  the anonymous user
```bash
addgroup --gid 6000 nfs
adduser --gid 6000 --uid 6000 nfs

delgroup nfs
deluser nfs
```
#### info
```bash
exportfs -a # export all
exportfs -r
exportfs -u
```
#### mounting remote folders
```bash
mount -t nfs  10.228.67.42:/srv/share1 /mnt/share1
mount -t nfs  -o ro 10.228.67.42:/srv/share1 /mnt/share1
```
#### editing /etc/fstab
```bash
vim /etc/fstab
```
```bash
10.10.2.124:/srv/share1 /mnt/share1 nfs defaults    0   0
```
## Samba
```bash
apt-get install samba smbclient cifs-utils
```
#### creating a folder to share
```bash
mkdir /srv/shared1
chown nfs:nfs shared1

chmod 777 /srv/shared1
```
#### creating a samba user off a system user
```bash
addgroup --gid 7000 smbshare
adduser --gid 7000 --uid 7000 smbshare
smbpasswd -a smbshare
```
```bash
pdbedit -w -L
```
#### using the smb client (not often used)
```bash
smbclient //localhost/share1 -U smbshare
smbclient -L localhost
```
#### conifuration /etc/samba/smb.conf
```bash
vim /etc/samba/smb.conf
```
```bash
[global]
        interfaces = 192.168.83.143/24 127.0.0.1/32
        bind interfaces only = yes

[share1]
        comment = Prvni share co jsem kdy vyrobili
        path = /srv/share1
        browsable = yes
        writable = yes
        guest ok = yes
        create mask = 0600
        directory mask = 0700

[share2]
        path = /srv/share2
        valid users = @users
        read only = yes
        
[share3]
        path = /srv/share3
        valid users = smbshare
        
[anonymous]
        path = /srv/anonymous
        force group = cifs
        create mask = 0660
        directory mask = 0771
        browsable =yes
        writable = yes
        guest ok = yes
```
```bash
/etc/init.d/smbd restart
```
#### mounting the shared folder
```bash
mount -t cifs //localhost/share1 /mnt/shared1samba -o username=smbshare
mount -t cifs //10.10.2.130/share1 /mnt/shared1samba -o username=smbshare
```

#### adding it to /etc/fstab
```bash
//10.10.2.130/share1	/mnt/shared1samba cifs username=smbshare,password=smbshare	0	0
```
