
#Unix Tips
**systemctl status application**

	systemctl status nginx
	 nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-05-15 04:39:50 EDT; 2 days ago
       Docs: man:nginx(8)
    Process: 495 ExecStartPre=/usr/sbin/nginx -t -q -g
Look at the line with **"Active"**.

Reverse proxy using nginx:
After configuring nginx and lets enc
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMzEwNTQwNTAsLTgzOTYwNjgwMF19
-->