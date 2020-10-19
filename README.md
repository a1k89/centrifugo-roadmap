# centrifugo-roadmap
Roadmap centrifugo deploy

https://centrifugal.github.io/centrifugo/

1. Install centrifugo server:
```
curl -s https://packagecloud.io/install/repositories/FZambia/centrifugo/script.deb.sh | sudo bash
```
2. Make config:
```
centrifugo genconfig
```
Change admin settings (if you wish) and password for admin interface.\

3. /etc/supervisor/conf.d/centrifugo.conf
```
[program:centrifugo-server]
command=centrifugo --config=/<home_folder>/config.json --admin
user=<user_here>

sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart all
```

4. /etc/nginx/sites-available
```
upstream centrifugo {
    ip_hash;
    server 127.0.0.1:8000;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}


server {

    server_name centrifugo.example.com;

    listen 80;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1000;
    gzip_proxied any;

    # Only retry if there was a communication error, not a timeout
    # on the Tornado server (to avoid propagating "queries of death"
    # to all frontends)
    proxy_next_upstream error;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_set_header Host $http_host;

    location /connection {
        proxy_pass http://centrifugo;
        proxy_buffering off;
        keepalive_timeout 65;
        proxy_read_timeout 60s;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

    location / {
        proxy_pass http://centrifugo;
    }

    error_page   500 502 503 504  /50x.html;

    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

sudo service nginx restart


Go to the http://centifugo.example.com and type password
```



