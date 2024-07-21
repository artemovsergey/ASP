# Docker-compose: .net, pg, pgadmin

```yml
version: '1.0'

networks:
  asp-dotnet-network:
    driver: bridge

services:
  api:
    container_name: ContainerAPI
    restart: always
    image: ${DOCKER_REGISTRY-}exampleapi
    build:
      context: .
      dockerfile: Example.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5000:5001"
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


# Nginx

```
worker_processes 4;
events { worker_connections 1024; }
http {
    server {
        listen 80;
        location /
        {
         proxy_pass http://client:3000;
        }

        location /api/users 
        {
         proxy_pass http://backend:80/api/user/users;
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


