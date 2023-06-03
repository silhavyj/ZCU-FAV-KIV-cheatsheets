# DNS
```bash
apt-get install bind9 dnsutils -y
```
#### defining zones (with views)
```bash
vim /etc/bind/named.conf.local
```
``` bash
view "private" {
        match-clients { 147.228.67.0/24; };
        recursion yes;

        zone "silhavyj.spos." {
                type master;
                file "/etc/bind/db.silhavyj.spos";
                allow-transfer {
                        147.228.67.0/24;
                };
        };

        zone "lucka.spos." {
                type slave;
                file "/var/cache/bind/db.lucka.spos";
                masters {
                        147.228.67.52;
                };
        };
        
        zone "67.228.147.in-addr.arpa" {
                type master;
                file "/etc/bind/db.silhavyj.spos.45";
        };
};

view "public" {
        match-clients { "any"; };
        recursion no;

        zone "silhavyj.spos." {
                type master;
                file "/etc/bind/public/db.silhavyj.spos";
        };
};
```
#### modifying default-zones because of views
```bind
vim /etc/bind/named.conf.default-zones
```
```bash
view "default" {
        match-clients {"any"; };
        .
        .
};
```
#### transferring and storing the zone file from the slave NS
``` bash
mkdir /etc/bind/slaves
chown -R bind:bind /etc/bind/slaves
// TODO doesn't work
```
#### creating a zone file
``` bash
cp /etc/bind/db.local /etc/bind/db.silhavyj.spos
vim /etc/bind/db.silhavyj.spos
```
``` bash
;
; BIND data file for local loopback interface
;
$TTL    3600
@       IN      SOA     silhavyj.spos. root.silhavyj.spos. (
                              12        ; Serial
                          86400         ; Refresh
                           3600         ; Retry
                          86400         ; Expire
                           3600 )       ; Negative Cache TTL
;
@       IN      NS      ns1.silhavyj.spos.
@       IN      NS      ns1.lucka.spos.
@       IN      A       147.228.67.45
@       IN      AAAA    fe80::46d2:1fff:fe31:a005
ns1     IN      A       147.228.67.45
ns1     IN      AAAA    fe80::46d2:1fff:fe31:a005
www     IN      CNAME   silhavyj.spos.
www2    IN      CNAME   lucka.spos.
;*      IN      A       147.228.67.45
@       IN      MX      20 mail.silhavyj.spos.
@       IN      MX      30 mail-backup.silhavyj.spos.
@       IN      MX      40 mail.lucka.spos.
mail    IN      A       147.228.67.45
mail-backup     IN      A       147.228.67.52
@       IN      TXT     "hello world"
```
#### creating a reverse zone file
```bash
cp /etc/bind/db.127 /etc/bind/db.silhavyj.spos.123
vim /etc/bind/db.silhavyj.spos.123
```
```bash
;
; BIND reverse data file for local loopback interface
;
$TTL    3600
@       IN      SOA     ns1.silhavyj.spos-test.spos. root.localhost. (
                              2         ; Serial
                          86400         ; Refresh
                           3600         ; Retry
                          86400         ; Expire
                           3600 )       ; Negative Cache TTL
;
@       IN      NS      ns1.
123     IN      PTR     ns1.silhavyj.spos-test.spos.
123     IN      PTR     mail.silhavyj.spos-test.spos.
123     IN      PTR     silhavyj.spos-test.spos.
123     IN      PTR     server.silhavyj.spos-test.spos.
123     IN      PTR     ssl.silhavyj.spos-test.spos.
```
#### restarting the service
```bash
/etc/init.d/bind9 restart
/etc/init.d/bind9 status
```
