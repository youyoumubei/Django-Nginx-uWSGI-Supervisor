# Django-Nginx-uWSGI-Supervisor
Build a Django web application on linux

## what's need
#### install
```
yum -y install epel-release
yum -y install python-django
yum -y install nginx
yum -y install gcc
pip install uwsgi
pip install supervisor
```
### disable firewalld etc..
```
systemctl disable firewalld.service

vim /etc/selinux/config
SELINUX=disabled
```
## test your uwsgi
### run test.py with command
```
uwsgi --http :9999 --wsgi-file /www/test.py
```
#### advanced options
```
uwsgi --http :9999 --wsgi-file /www/test.py --processes=4 --threads=2
```
## deploying django project with nginx+uwsgi+supervisor
### nginx configuration
```
server {
        #listening port
    	listen         8080;
	    server_name    127.0.0.1;
	    charset UTF-8;
	    access_log      /var/log/nginx/mysite_access.log;
	    error_log       /var/log/nginx/mysite_error.log;

	    client_max_body_size 75M;

        #uwsgi configuration
	    location / {
		    include uwsgi_params;
		    uwsgi_pass 127.0.0.1:9090;
		    uwsgi_read_timeout 2;
	    }

        #static source path
	    location /static {
		    expires 30d;
		    autoindex on;
		    add_header Cache-Control private;
		    alias /var/www/mysite/static/;
	    }
    }
```
### uwsgi configuration
```
[uwsgi]
#指定项目目录
chdir = /path/mysite
#wsgi module
module          = mysite.wsgi
master          = true
#进程数
processes       = 1
#port
socket     = :9090
vacuum          = true 

```
### supervisor configuration
```
[program:uwsgi9090]
command = /usr/bin/uwsgi --ini /etc/uwsgi.ini
autorestart = true

redirect_stderr = true
stdout_logfile = /var/log/mysite.log
stopsignal = INT
```
### run
```
systemctl start nginx.service
systemctl enable nginx.service

supervisorctl start uwsgi9090
```