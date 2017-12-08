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

## run falcon using werkzeug

assuming this is your `main.py` file

```python
import falcon

class QuoteResource:
    def on_get(self, req, resp):
        """Handles GET requests"""
        quote = {
            'quote': (
                "I've always been more interested in "
                "the future than in the past."
            ),
            'author': 'Grace Hopper'
        }

        resp.media = quote

app = falcon.API()
app.add_route('/quote', QuoteResource())
```
Create a runserver.py file and add the code below
```python
from main import falcon_app

if __name__ == '__main__':
	from wsgiref.simple_server import make_server
	httpd = make_server('0.0.0.0', 8081, falcon_app)
	httpd.serve_forever()
```
    
