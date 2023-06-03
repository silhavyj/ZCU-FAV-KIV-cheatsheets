# File systems
#### basic commands
```bash
lsblk
blkid
df -h
du -hs /*
mkfs.ext4 /dev/md0
mkfs.xfs /dev/sdc
mkswap /dev/sda3
swapon /dev/sda3
```
#### mounting
```bash
mount /dev/md0 /mnt/md0_storage
umount /mnt/md0_storage
mount -a
e2label /dev/sdb1 "label"
vim /etc/fstab
```
#### installing xfs file system
```bash
apt search xfs
```
```bash
apt-get install xfsprogs -y
```
#### images
```bash
dd if=/dev/zero of=/tmp/image bs=1M count=1024
losetup -f /tmp/image
losetup -D /tmp/image
mkfs.ext4 /dev/loop0
wipefs -a /dev/loop0 
mount  /dev/loop0 /mnt/
```
#### creating partitions
```bash
fdisk /dev/sdb
```
## mdadm
```bash
apt-get install mdadm -y
```
```bash
raid0 - linear / striping
raid1 - mirroring
raid5 - striping + parita
raid6 - striping + 2x parita

raid01 - raid0 + raid1
raid10 - raid1 + raid0
raid50 - raid5 + raid0
raid60 - raid6 + raid0
```
```bash
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sd[b-c]
mdadm -Cv /dev/md0 -l1 -n2 /dev/sd[b-c]1
mdadm -Cv /dev/md0 -l5 -n3 /dev/sd[b-d]1
mdadm -Cv /dev/md0 -l6 -n4 /dev/sd[b-d]1 missing

cat /proc/mdstat
watch --interval=10 cat /proc/mdstat

mdadm --stop /dev/md0
mdadm --remove /dev/md0
mdadm --detail /dev/md0
mdadm --fail /dev/md0 /dev/sda1
mdadm --add /dev/md0 /dev/sdc1
```
```bash
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
blkid | grep /dev/md0 | awk '{print $2}' >> /etc/fstab
```
## lvm
```bash
apt-get install lvm2 -y
```
#### physical volume
```bash
pvs
pvdisplay | pvscan | pvremove
pvcreate /dev/md0
pvmove -v /dev/sda2 /dev/sdb2
```
#### volume groups
```bash
vgs
vgdisplay | vgscan | vgchange | vgremove
vgcreate vg-data /dev/md0
vgextend vg-data /dev/md1
vgreduce vg-data /dev/md0

vgreduce --removemissing --verbose myVG_NAME
```
#### logical volume
```bash
lvs
lvdisplay | lvscan | lvchange | lvresize | lvrename
lvcreate -L 30G -n db-data vg-data
lvextend -L +100MB -n /dev/data/database-mysql /dev/md127

resize2fs /dev/vg-data/db-data

lvcreate -L 30G -m1 -n db-data vg-data
lvcreate --size 100M --snapshot -n db-data-snap /dev/vg-data/db-data
lvremove /dev/vg-data/db-data-snap
```
```bash
mount /dev/vg-data/db-data /mnt/data1
mount /dev/vg-data/db-data-snap /mnt/data2
```
## /etc/fstab
```bash
/dev/data/usr   /usr    ext4    defaults        0       0
/dev/data/database-mysql        /mnt/databases/mysql    ext4    defaults        0       0
/dev/data/database-postgres     /mnt/databases/postgres ext4    defaults        0       0
/mnt/databases/mysql    /var/lib/mysql  bind    bind    0       0
/mnt/databases/postgres /var/lib/postgresql     bind    bind    0       0
LABEL=backup    /mnt/backup     ext4    defaults        0       0
LABEL=test      /mnt/test       ext4    ro      0       0
```
