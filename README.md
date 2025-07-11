# Docker Nginx proxy

Hosts Configuration
-------
Edit the `nginx.conf` file with your own server block(s).

Always listen on port 80 and/or 443 and proxy pass this to your Docker container port:
```
server {
    listen 80;
    server_name my-project.loc;

    location / {
        proxy_pass http://host.docker.internal:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Put your domain in your hosts file (Mac: `/etc/hosts`):
```
127.0.0.1   my-project.loc
```

HTTPS/SSL Configuration
-------
Generate a certificate for your domain:
```
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout certs/my-project.loc.key \
  -out certs/my-project.loc.crt \
  -subj "/C=NL/ST=Dev/L=DevCity/O=DevOrg/OU=devOrgUnit/CN=my-project.loc"
```
Edit your `nginx.conf` server block as follows:
```
server {
    listen 80;
    server_name my-project.loc;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name my-project.loc;

    ssl_certificate         /etc/nginx/certs/my-project.loc.crt;
    ssl_certificate_key     /etc/nginx/certs/my-project.loc.key;
    ssl_protocols           TLSv1.2 TLSv1.3;

    ssl_prefer_server_ciphers off;

    location / {
        access_log off;

        proxy_pass       http://host.docker.internal:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

Docker Container
---------
Start the nginx-proxy container with:
```
docker-compose up -d
```
Stop your container with:
```
docker-compose down
```
~ Remember to always restart your container if your change your configurations
