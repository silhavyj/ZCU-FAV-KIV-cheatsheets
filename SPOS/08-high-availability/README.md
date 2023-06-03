
  
## DNS round robin
```bash
www	IN	A	147.228.67.42
www	IN	A	147.228.67.43
www	IN	A	147.228.67.44
```
## NginX
The port Apache is running on should be first change to 8080 or something. Otherwise, `nginx` installation won't finish as is uses the same ports as Appache (80).

```bash
apt-get install nginx
```
```bash
vim /etc/nginx/sites-enabled/default
```
```bash
upstream apache  {
        # ip_hash;
        server localhost:8080 max_fails=2 fail_timeout=5s;
        server localhost:8081 max_fails=2 fail_timeout=5s;
}

server {
    listen 81;
    server_name _;

    location / {
        proxy_pass http://apache;
    }

    location /img {
        proxy_pass http://localhost:8080/img/;
    }
}

server {
    listen 82;
    server_name _;

    location / {
        proxy_pass http://apache;
    }

    location /img {
        proxy_pass http://localhost:8081/img/;
    }
}

server {
    listen 80;
    server_name test.spos www.test.spos;
    return 301 https://ssl.test.spos$request_uri;
}
```
```bash
nginx -t # checks the configuration
/etc/init.d/nginx restart
```
#### testing
```bash
mkdir /var/www/web01.silhavyj.spos-test.spos/img
echo web01-img > /var/www/web01.silhavyj.spos-test.spos/img/index.html
```
```bash
curl localhost:81
curl localhost:81/img
curl localhost:82
curl localhost:82/img
```
