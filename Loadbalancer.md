# Контейнер Nginx

```
FROM nginx:alpine
RUN apk add --no-cache openssl

# SSL
RUN mkdir -p /etc/nginx/ssl

COPY ./certificate/localhost.crt /etc/nginx/ssl/localhost.crt
COPY ./certificate/localhost.key /etc/nginx/ssl/localhost.key
COPY ./certificate/localhost_full.crt /etc/nginx/ssl/localhost_full.crt

#RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx-selfsigned.key -out /etc/nginx/ssl/nginx-selfsigned.crt -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=localhost"
#RUN openssl req -x509 -newkey rsa:4096 -keyout localhost.key -out localhost.crt -days 365 -nodes

COPY ./loadbalancer/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]

```

# nginx.conf

```
  events {
    worker_connections 1024;
  }
http {
  upstream angular {
    server angular:4200;
  } 
  upstream api {
    server api:5001;
  }
  

 server {
    listen 80;
    listen 443 ssl;
    #return 301 https://$host$request_uri;
    server_name angular;
    server_name api;

    # https
    ssl_certificate /etc/nginx/ssl/localhost.crt;
    ssl_certificate_key /etc/nginx/ssl/localhost.key;
    ssl_trusted_certificate /etc/nginx/ssl/localhost_full.crt;

    #ssl_certificate /etc/letsencrypt/live/$host/fullchain.pem;
    #ssl_certificate_key /etc/letsencrypt/live/$host/privkey.pem;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
       
       proxy_pass http://angular;
       # протокол websocket
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "Upgrade";
       proxy_set_header Host $host;
    }
    location /api {
  
       proxy_pass https://api;
       proxy_set_header Host $host;
    }
  }
}

```
