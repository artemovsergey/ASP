# DockerCompose

docker-compose.yml
```yml
version: '2'

services:

  nginx:
    image: nginx:mainline
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"

  client:
    build:
      context: ./my-app
      dockerfile: dockerfile
    ports:
      - "5001:3000"

  backend:
    restart: always
    build:
      context: .
      dockerfile: dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5000:80"



```
