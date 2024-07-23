# Команды 

- docker network ls
- docker network inspect example_asp-dotnet-network

Замечание: имена сервисов Docker работают только с кодом, запущенным в сети контейнеров


# Docker-compose: nginx,angular,.net, pg, pgadmin

```yml
version: '1.0'

networks:
  asp-dotnet-network:
    driver: bridge

services:

  nginx:
    container_name: ContainerNginx
    build: 
      context: .
      dockerfile: loadbalancer/Dockerfile
    restart: always
    ports:
      - "80:80"
    links:
      - api
      - angular
    depends_on:
      - db
      - api
      - angular
    networks:
      - asp-dotnet-network 

  angular:
    container_name: ContainerAngular
    restart: always
    build:
      context: .
      dockerfile: Example.Angular/Dockerfile
    environment:
      - Production
    ports:
      - "4200:4200"
    networks:
      - asp-dotnet-network
    depends_on:
      - db
      - api
  api:
    container_name: ContainerAPI
    restart: always
    build:
      context: .
      dockerfile: Example.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5010:5001"
    networks:
      - asp-dotnet-network
    depends_on:
      - db
  db:
    image: postgres:latest
    container_name: ContainerPostgreSQL
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: root
      POSTGRES_DB: Example
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - asp-dotnet-network
  pgadmin:
    container_name: ContainerPgAdmin
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: root
    volumes:
      - postgres_data:/var/lib/postgresql/dbadmin
    ports:
      - "5050:80"
    networks:
      - asp-dotnet-network 

volumes:
    postgres_data:


```

# nginx.conf

```
  events {
    worker_connections 1024;
  }
http {
  upstream angular {
    # These are references to our backend containers, facilitated by
    # Compose, as defined in docker-compose.yml
    server angular:4200;
  } 
  upstream api {
    # These are references to our backend containers, facilitated by
    # Compose, as defined in docker-compose.yml
    server api:5001;
  }
  

 server {
    listen 80;
    server_name angular;
    server_name api;

    location / {
       resolver 127.0.0.11 valid=30s;
       proxy_pass http://angular;
       # для работы websocket
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "Upgrade";
       proxy_set_header Host $host;
    }
    location /api {
       resolver 127.0.0.11 valid=30s;
       proxy_pass https://api;
       proxy_set_header Host $host;
    }
  }
}

```

- docker compose up --build
- docker compose up --build -d
- docker compose down
- git stash -u  сохраняет все изменения перед созданием новой ветки
- docker container ls  - список все запущенных контейнеров
- docker exec -it 39fdcf0aff7b bash  - запуск команды в контейнере по id контейнера
- example=# INSERT INTO "Students" ("ID", "LastName", "FirstMidName", "EnrollmentDate") VALUES (DEFAULT, 'Whale', 'Moby', '2013-03-20');
- docker compose rm
- docker compose up --build
- docker compose watch


