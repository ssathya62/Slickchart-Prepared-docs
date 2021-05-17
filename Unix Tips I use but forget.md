
#Unix Tips
**systemctl status application**

	systemctl status nginx
	 nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-05-15 04:39:50 EDT; 2 days ago
       Docs: man:nginx(8)
    Process: 495 ExecStartPre=/usr/sbin/nginx -t -q -g
Look at the line with **"Active"**.

**Reverse proxy using nginx**:
After configuring nginx, lets encrypt and setup your reverse proxy as follows:

 - Locate the file default in folder /etc/nginx/sites-available
 - In section server add the following at the appropriate place.

```sh
server {

        #root /var/www/html;

         root /home/srvean/application/TW-Scheduler/TW-Scheduler/bin/Release/net5.0;
          location / {
                proxy_pass https://localhost:5001;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection keep-alive;
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_buffer_size       128k;
                proxy_buffers           4 256k;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/tailboy.tk/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/tailboy.tk/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot



}
```
Before restarting nginx test if configuration is good by using the command 

	nginx -t
	nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
	nginx: configuration file /etc/nginx/nginx.conf test is successful
Make sure everything is find before we go ahead.
Now create a new file in folder */etc/systemd/system*. For convenience if your application name is GreatApp create a file GreatApp.service in this folder and have its content along these lines:
```sh
[Unit]
Description= StockSelector
[Service]
User=srvean
Group=srvean
WorkingDirectory=/home/srvean/DotNet/bin/StockSelector
ExecStart=/home/srvean/DotNet/bin/StockSelector/StockSelector.Server
Restart=always
# Restart service after 60 seconds if the dotnet service crashes:
RestartSec=60
SyslogIdentifier=StockSelector
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```
Now enable the service with the command
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU1MDQ5NjE3OSwtMzE5NzU5MDU2LC0zOD
Y2MTk1MjYsLTgzOTYwNjgwMF19
-->