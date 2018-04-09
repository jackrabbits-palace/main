# nginx reverse proxy
since our dns records for any domain we wish to host on a machine(s) on our LAN will all point to the same ip address (the public ip address of our router) on port 80, we want to setup a reverse proxy server which our router forwards all incoming traffic to. The reverse proxy should be able to use the `host` field within any request to direct traffic to the appropriate location within the LAN. we will use nginx for this since it is pretty straight forward to setup.

### install and run nginx on pi
```shell
$ sudo apt-get install nginx
$ sudo systemctl enable nginx
$ sudo systemctl start nginx
```

### edit the nginx configuration
edit the `/etc/nginx/conf.d/default.conf`,
```shell
## Basic reverse proxy server ##

server_names_hash_bucket_size 100;

## someservice service on slime.church ##
upstream someservice  {
      server 127.0.0.1:3141; #someservice
}

## Start slime.church static landing page ##
server {
    listen       80;
    server_name  slime.church;

    access_log  /var/log/nginx/slime.church.access.log  combined;
    error_log  /var/log/nginx/slime.church.error.log;
    root   /home/pi/Public;
    index  index.html index.htm;
}
## End slime.church ##

## START someservice.slime.church ##
server {
       listen      80;
       server_name someservice.slime.church;
       access_log  /var/log/nginx/someservice.slime.church.access.log  combined;
       error_log   /var/log/nginx/someservice.slime.church.error.log;

       location / {
                proxy_pass  http://someservice;
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
                proxy_redirect off;
                proxy_buffering off;
                proxy_set_header        Host            someservice.slime.church;
                proxy_set_header        X-Real-IP       $remote_addr;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```
this configuration defines the directory for a static site to be served for `slime.church` and also a dynamic server to be proxied to for `someservice.slime.church`.

# SSL with Let's Encrypt
we don't want traffic to our sites to be cleartext so we need to get an SSL certificate to enable HTTPS. **Let's Encrypt** is a free Certificate Authority (CA), so let's use that.

we will use `Certbot` to automate certificate issuance and installation.

### install `Certbot`
if you go to the [Certbot](https://certbot.eff.org/) site it will tell you how to install and use it,
```shell
$ sudo apt-get install python-certbot-nginx
```

### port forward router port 443 to pi port 443
again, log into your router's web console and port forward.

### obtain certificates and modifying nginx configuration to serve certs
at the time of writing this, the way to obtain certificates is the following (this will handle obtaining certs and modifying the nginx configuration to serve them),
```shell
sudo certbot --authenticator standalone --installer nginx --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
```
this will stop and restart your nginx server while obtaining certs using the pre/post-hooks. you will be asked some questions which are easy to answer. certs will be save in `/etc/letsencrypt/live/slime.church/fullchain.pem`.

### auto renew certificates
since certs last for only 90 days, we want to continuously try to renew our certs if we can. `certbot` ships with a systemd service to easily handle auto-renew
```shell
$ sudo systemctl enable certbot.timer
$ sudo systemctl start certbot.timer
```

ok. we should be good for now.
