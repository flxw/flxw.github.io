---
layout: post
title: Making nginx act as an SMTP relay
---

To test an application out in the wild I had to host it on my server.
As it receives and processes email via SMTP, it needed access to port 25.

I did not bother learning and installing some tool that makes sub-1000 ports
available to applications without superuser rights, and thus I made use
of nginx' capabilities.

Inside the main *nginx.conf*, I added a section for email:

```
mail {
    # See sample authentication script at:
    # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript

    # auth_http localhost/auth.php;
    # pop3_capabilities "TOP" "USER";
    # imap_capabilities "IMAP4rev1" "UIDPLUS";

    auth_http http://127.0.0.1/mail/auth;
    xclient off;
    proxy_pass_error_message on;

    server {
        listen     25;
        protocol   smtp;
        timeout    5s;
        proxy      on;
        xclient    off;
        smtp_auth  none;
        so_keepalive on;
    }
}
```

Each of the configurations for my sites are loaded as individual modules, like it is the case with Apache.
Inside the configuration that handles the default behaviour for the web server, the following route was established.
It tells the mail client where to find the SMTP server. This can also be equipped with more logic to make it act as a load balancer.

```
    location = /mail/auth {
            set $reply ERROR;

            if ($http_auth_smtp_to ~ flxw.de) {
                set $reply OK;
            }

            add_header Auth-Status OK;
            add_header Auth-Server 127.0.0.1;
            add_header Auth-Port 2525;
            add_header Auth-Wait 1;
            return 204;
        }
```

And finally there is the configuration file for the web app itself.

```
upstream app.myapp.de {
    server 127.0.0.1:8080;
}

# the nginx server instance
server {
    listen 0.0.0.0:80;
    server_name myapp.flxw.de;
    access_log /var/log/nginx/myapp.mydomain.de.log;

    auth_basic  "Restricted";
    auth_basic_user_file /etc/nginx/htpasswd;

    # pass the request to the node.js server with the correct headers and much more can be added, see nginx config options
    location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy true;

      proxy_pass http://myapp.mydomain.de/;
      proxy_redirect off;
    }
 }
```