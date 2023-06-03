# Apache
```bash
apt-get install apache2 curl
```
#### enable/disable a site
```bash
a2ensite www.silhavyj.spos
a2dissite www.silhavyj.spos
```
#### enable/disable a module
```bash
a2enmod ssl
a2dismod ssl
```
#### restarting the service
```bash
apachectl -t # syntax
apachectl -S # active clients	
```
```bash
/etc/init.d/apache2 restart
/etc/init.d/apache2 reload
```
#### creating a virtual host
```bash
cd /etc/apache2/sites-available
cp 000-default.conf www.silhavyj.spos.conf
vim www.silhavyj.spos.conf
```
```bash
<VirtualHost *:80>
        ServerName www.silhavyj.spos
        ServerAlias silhavyj.spos
        ServerAdmin silhavyj@spos
        
        DocumentRoot /var/www/www.silhavyj.spos
	
        Redirect / http://www.tonda.spos

        Alias /pma /usr/share/phpmyadmin
	
	<Directory />
                Options FollowSymLinks
                AllowOverride None
		Require all granted
        </Directory>
        
        <Location /secret>
                AuthType Basic
                AuthName "Restricted Access"
                AuthUserFile /etc/apache2/.htpasswd
                Require user silhavyj ryan
        </Location>
	
	<Directory /var/www/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/www.silhavyj.spos-error.log
        CustomLog ${APACHE_LOG_DIR}/www.silhavyj.spos-access.log combined

	# Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn
</VirtualHost>
```
#### creating a root directory for the virtual host
```bash
mkdir -p /var/www/www.silhavyj.spos
echo 'ServerName: www.silhavyj.spos' > /var/www/www.silhavyj.spos/index.html
a2ensite www.silhavyj.spos
systemctl reload apache2
```
```bash
curl -H "Host: www.silhavyj.spos-test.spos" http://silhavyj.spos-test.spos:8888
```
```bash
for i in `seq 1 10`; do touch file_$i; done
```
#### creating users for restricted access
```bash
htpasswd -c /etc/apache2/.htpasswd silhavyj # for the first time
htpasswd -B /etc/apache2/.htpasswd ryan     # adds another user
```
```bash
curl www.silhavyj.spos/secret/ -u "silhavyj:123456"
```
#### disabling showing the version and the OS when using curl -v
```bash
vim /etc/apache2/conf-available/security.conf
ServerTokens Prod
```
#### changing the listening port
```bash
 vim /etc/apache2/ports.conf
```
```bash
#Listen 80
Listen 127.0.0.1:8888

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```
```bash
<VirtualHost *:8888>
.
.
.
</VirtualHost>
```
## PHP
```bash
apt search libapache2-mod-php
apt-get install libapache2-mod-php7.3
```
#### configuration
```bash
vim /etc/php/7.3/apache2/php.ini
```
```bash
upload_max_filesize = 16M
memory_limit = 128M
max_input_time = 60
post_max_size = 16M
max_execution_time = 30

open_basedir = ...
disable_functions = ... phpinfo
disable_classes = ... 
allow_url_fopen = On/Off
allow_url_include = On/Off
display_errors = On/Off
error_reporting = E_ALL
log_errors = On/Off
```
#### index.php
```php
<?php
    $page_title = date("Y-m-d H:i:s");
    echo("<title>$page_title</title>");
    phpinfo();
?>
```
#### test.php
```php
// http://silhavyj.spos/index.php?page=/etc/passwd
<?php
    $page = $_GET['page'];
    $xfile = fopen($page, "r") or die("Unable to open file!");
    echo fgets($xfile);
    fclose($xfile);
?>
```
### display php version
```php
<?php
    echo 'current php version: ' . phpversion();
    echo "\n";
?>
```
## Self-signed certificate
#### creating a directory for certificates
```bash
mkdir /etc/apache2/ssl
cd /etc/apache2/ssl
```
#### creating and signing a new certificate
```bash
openssl genrsa -des3 -out server.key 4096
cp server.key server.key.org
openssl req -new -key server.key -out server.csr
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
#### modifying the default-ssl virtual host
```bash
vim /etc/apache2/sites-available/default-ssl.conf
```
```bash
<IfModule mod_ssl.c>
        <VirtualHost *:443>
                ServerAdmin silhavyj@spos
                ServerName www.silhavyj.spos

                DocumentRoot /var/www/ssl-www.silhavyj.spos

                ErrorLog ${APACHE_LOG_DIR}/ssl-www.silhavyj.spos.error.log
                CustomLog ${APACHE_LOG_DIR}/ssl-www.silhavyj.spos.access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/apache2/ssl/server.crt
                SSLCertificateKeyFile   /etc/apache2/ssl/server.key

                #SSLCertificateChainFile /etc/apache2/ssl.crt/server-ca.crt

                #SSLCACertificatePath /etc/ssl/certs/
                #SSLCACertificateFile /etc/apache2/ssl.crt/ca-bundle.crt

                #SSLCARevocationPath /etc/apache2/ssl.crl/
                #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl

                #SSLVerifyClient require
                #SSLVerifyDepth  10

                #SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>
        </VirtualHost>
</IfModule>
```
#### creating a root directory for the virtual host
```bash
mkdir -p /var/www/ssl-www.silhavyj.spos
echo 'self-signed HTTPS: www.silhavyj.spos' > /var/www/ssl-www.silhavyj.spos/index.html
```
#### enabling the ssl module and restarting the service
```bash
a2enmod ssl
a2ensite default-ssl.conf
systemctl reload apache2
```

## Lets encrypt
```bash
apt-get install git
```
```bash
cd /opt && git clone https://github.com/lukas2511/dehydrated && cd dehydrated
mkdir /etc/dehydrated/
cp ./docs/examples/hook.sh /etc/dehydrated/hook.sh && chmod +x /etc/dehydrated/hook.sh
```
#### creating a configuration
```bash
vim /etc/dehydrated/config
```
```bash
DOMAINS_TXT=/etc/dehydrated/domains
WELLKNOWN="/var/www/dehydrated/.well-known/acme-challenge/"
CERTDIR="/etc/letsencrypt/live/"
CONTACT_EMAIL=silhavy@students.zcu.cz
HOOK=/etc/dehydrated/hook.sh
```
```bash
mkdir -p /var/www/dehydrated/.well-known/acme-challenge/
echo "spos-05.kiv.zcu.cz" > /etc/dehydrated/domains
```
#### telling hook.sh to reload apache2
```bash
vim /etc/dehydrated/hook.sh
```
```bash
deploy_cert() {
	.
	.
	systemctl reload apache2
}
```
#### creating a virtual host with a public domain spos-05.kiv.zcu.cz
```bash
cd /etc/apache2/sites-available/
cp default-ssl.conf spos-05.kiv.zcu.cz-ssl.conf
vim spos-05.kiv.zcu.cz-ssl.conf
```
```bash
<IfModule mod_ssl.c>
        <VirtualHost *:443>
                ServerAdmin webmaster@localhost
                ServerName spos-05.kiv.zcu.cz

		Alias /.well-known/acme-challenge/ /var/www/dehydrated/.well-known/acme-challenge/

                DocumentRoot /var/www/spos-05.kiv.zcu.cz

                ErrorLog ${APACHE_LOG_DIR}/ssl-spos-05.kiv.zcu.cz.error.log
                CustomLog ${APACHE_LOG_DIR}/ssl-spos-05.kiv.zcu.cz.access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/letsencrypt/live/spos-05.kiv.zcu.cz/cert.pem
                SSLCertificateKeyFile  /etc/letsencrypt/live/spos-05.kiv.zcu.cz/privkey.pem
                SSLCertificateChainFile  /etc/letsencrypt/live/spos-05.kiv.zcu.cz/chain.pem

                #SSLCACertificatePath /etc/ssl/certs/
                #SSLCACertificateFile /etc/apache2/ssl.crt/ca-bundle.crt

                #SSLCARevocationPath /etc/apache2/ssl.crl/
                #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl

                #SSLVerifyClient require
                #SSLVerifyDepth  10

                #SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>
        </VirtualHost>
</IfModule>
```
#### accepting the terms and asking for a certificate
```bash
./dehydrated --register --accept-terms
./dehydrated --cron
```
#### enabling the site and reloading apache
```bash
a2ensite
spos-05.kiv.zcu.cz-ssl
systemctl reload apache2
```
