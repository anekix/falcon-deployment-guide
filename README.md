# falcon-deployment-guide
Deployment Guide for falcon

## Deployment with nginx & uwsgi

Nginx acts as the entry point of all requests to the server & then the request is passed to Uwsgi server which runs falcon app.

Nginx Conf
This is the bare minimum conf required for nginx


`File: nginx_project.conf`                                                                      

```nginx
server {
        listen *:80;
        client_max_body_size 4G;
        access_log /path_to_access_log;
        error_log /path_to_error_log;

        location /api
         {
                    root /path_to_falcon_app;
                    uwsgi_read_timeout 180;
                    uwsgi_pass unix:///tmp/fb_weaver.sock;
                    include uwsgi_params;
         }
}
```

`uwsgi conf`

```uwsgi
[uwsgi]
base  =  /path_to_falcon_app
baselog = /path_to_uwsgi_log/src_uwsgi.log
basesock = /tmp/falcon_project.sock
plugins = python
socket = %(basesock)
chdir = %(base)
module = main:app
processes = 4
threads = 1
master = 1
reload-on-as = 512
reload-on-rss = 192
max-requests = 5000
limit-as = 1024
cpu-affinity = 3
reload-mercy = 15
no-orphans
vacuum
logto2 = %(baselog)
logfile-chmod = 777
chmod-socket = 777
post-buffering = 1
```

