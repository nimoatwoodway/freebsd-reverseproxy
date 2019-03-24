# freebsd-reverseproxy
Reverseproxy in a freebsd jail with lets encrypt.
Create a new jail and set router portforwardings for port 80 and 443 to the new reverseproxy jail.

## Install Packages
```
pkg update && pkg upgrade
pkg install nginx git vim py27-certbot-nginx py27-certbot-dns-cloudflare
```

## Create Proxyforwardings in nginx.conf
```
vim /usr/local/etc/nginx/nginx.conf
```
### Add a new server entry
```
server {
  server_name sample.example.tld;
  location /.well-known {
    root /usr/local/www/sample.example.tld/;
   }
   location / {
     proxy_pass http://internal_host:8888;
   }
 }
 ```
 
 Location *** .well-known*** is needed for lets enrypt certificate creation.
 
 ### Create Lets Encrypt certificate for new forwarding
 ```
 mkdir -p /usr/local/www/sample.example.tld/.well-known
 certbot --nginx -d sample.example.tld
 ```
 If everything works you will get asked how to redirect http traffic. For safety I choose ***2*** to redirect all traffic to https.
 
 ### Easy as Schnitzel isn't it?
