# freebsd-reverseproxy
Reverseproxy in a freebsd jail with lets encrypt. 

Have multiple web services whiche should be accessed with port 80 and 443 through one public IP address? The magic service you need herefor is a reverseproxy: https://en.wikipedia.org/wiki/Reverse_proxy

In my case for a FreeNas Setup where I have to redirect nextcloud and other services and having only one public IP.
Create a new jail and set router portforwardings for port 80 and 443 to the new reverseproxy jail.

## Install Packages
```
pkg update && pkg upgrade
pkg install nginx git vim py27-certbot py27-certbot-nginx
```

### Enable nginx on system start
```
echo 'nginx_enable="YES"' >> /etc/rc.conf
```

## Create Proxyforwardings in nginx.conf
```
vim /usr/local/etc/nginx/nginx.conf
```
### Add a new server entry
```
server {
  server_name sample.example.tld;
  
  # allow big file uploads with e.g. nextcloud
  client_max_body_size 10G;
  client_body_buffer_size 400M;
  
  location /.well-known {
    root /usr/local/www/sample.example.tld/;
   }
   location / {
     proxy_pass http://internal_host:8888;
   }
 }
 ```
 
 Location ***.well-known*** is needed for lets enrypt certificate creation.
 
 ## Create Lets Encrypt certificate for new forwarding
 ```
 mkdir -p /usr/local/www/sample.example.tld/.well-known
 certbot --nginx -d sample.example.tld
 ```
 On first run you will be asked to enter your email for renewal notifications.
 
 If everything works you will get asked how to redirect http traffic. For safety I choose ***2*** to redirect all traffic to https.
 
 ### Create cronjob for autorenewal
 ```
 crontab -e
 ```
 Add this line:
 ```
 52 0,12 * * * root /usr/local/bin/certbot renew --renew-hook 'service nginx reload'
 ```
 
 ## Example of adjusting nextcloud conf
 Probably you have to adjust the config of the service you are redirecting to a little bit. Here an example of lines to add to the nextcloud config file:
 ```
 vim /usr/local/www/nextcloud/config/config.php
 ```
 ```
 'trusted_proxies'   => ['ip_of_proxy_jail'],
 'overwritehost'     => 'sample.example.tld',
 'overwriteprotocol' => 'https',
 ```
 Add your internal nextcloud address to array of trusted_domains
 ```
 'trusted_domains' =>
  array (
    0 => 'sample.example.tld',
    1 => 'internal_host',
  ),
  ```
 
 ## Easy as Schnitzel isn't it?
