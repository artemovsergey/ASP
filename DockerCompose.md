# Dockerfile .Net 8

```
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["./SportStore.APIWithDocker.csproj", "./SportStore.APIWithDocker.csproj"]
RUN dotnet restore "./SportStore.APIWithDocker.csproj"
COPY . .

RUN dotnet build "./SportStore.APIWithDocker.csproj" -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./SportStore.APIWithDocker.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "SportStore.APIWithDocker.dll"]
```

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

```
version: '1'

networks:
  asp-dotnet-network:
    driver: bridge

services:
 
  api:
    restart: always
    build:
      context: ./SportStore.APIWithDocker
      dockerfile: Dockerfile
    container_name: ContainerAPI
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "3001:8080"
    networks:
      - asp-dotnet-network
    depends_on:
      - db
  db:
    image: postgres:latest
    container_name: ContainerPostgreSQL
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: MirtekNewsAggregation
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - asp-dotnet-network

  pgadmin:
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


# Docker for API

```docker
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["webapitest.csproj", "."]
RUN dotnet restore "webapitest.csproj"
COPY . .
RUN dotnet build "webapitest.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "webapitest.csproj" -c Release -o /app/publish /p:UseAppHost=false
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "webapitest.dll"]
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
# Dockerfile для .NET

```
# syntax=docker/dockerfile:1

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:6.0-alpine AS build
ARG TARGETARCH
COPY . /source
WORKDIR /source/src
RUN --mount=type=cache,id=nuget,target=/root/.nuget/packages \
    dotnet publish -a ${TARGETARCH/amd64/x64} --use-current-runtime --self-contained false -o /app

FROM mcr.microsoft.com/dotnet/sdk:6.0-alpine AS development
COPY . /source
WORKDIR /source/src
CMD dotnet run --no-launch-profile

FROM mcr.microsoft.com/dotnet/aspnet:6.0-alpine AS final
WORKDIR /app
COPY --from=build /app .
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser
USER appuser
ENTRYPOINT ["dotnet", "myWebApp.dll"]
```

# Dockerfile для тестов во время сборки

```
# syntax=docker/dockerfile:1

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:6.0-alpine AS build
ARG TARGETARCH
COPY . /source
WORKDIR /source/src
RUN --mount=type=cache,id=nuget,target=/root/.nuget/packages \
    dotnet publish -a ${TARGETARCH/amd64/x64} --use-current-runtime --self-contained false -o /app
RUN dotnet test /source/tests

FROM mcr.microsoft.com/dotnet/sdk:6.0-alpine AS development
COPY . /source
WORKDIR /source/src
CMD dotnet run --no-launch-profile

FROM mcr.microsoft.com/dotnet/aspnet:6.0-alpine AS final
WORKDIR /app
COPY --from=build /app .
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser
USER appuser
ENTRYPOINT ["dotnet", "myWebApp.dll"]
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

## Запуск тестов

- docker compose run --build --rm server dotnet test /source/tests
- docker build -t dotnet-docker-image-test --progress=plain --no-cache --target build .

# DockerCompose Postgres

```yml
services:
  server:
    build:
      context: .
      target: development
    ports:
      - 8080:80
    depends_on:
      db:
        condition: service_healthy
    develop:
      watch:
        - action: rebuild
          path: .
    environment:
       - ASPNETCORE_ENVIRONMENT=Development
       - ASPNETCORE_URLS=http://+:80'
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
```

# Dockerfile Angular

```
FROM node:14-alpine as build
WORKDIR /app

RUN npm install -g @angular/cli

COPY ./package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx as runtime
COPY --from=build /app/dist/client /usr/share/nginx/html
```

# Dockerfile Clean Architecture

```
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

# copy all the layers' csproj files into respective folders
COPY ["./ContainerNinja.Contracts/ContainerNinja.Contracts.csproj", "src/ContainerNinja.Contracts/"]
COPY ["./ContainerNinja.Migrations/ContainerNinja.Migrations.csproj", "src/ContainerNinja.Migrations/"]
COPY ["./ContainerNinja.Infrastructure/ContainerNinja.Infrastructure.csproj", "src/ContainerNinja.Infrastructure/"]
COPY ["./ContainerNinja.Core/ContainerNinja.Core.csproj", "src/ContainerNinja.Core/"]
COPY ["./ContainerNinja.API/ContainerNinja.API.csproj", "src/ContainerNinja.API/"]

# run restore over API project - this pulls restore over the dependent projects as well
RUN dotnet restore "src/ContainerNinja.API/ContainerNinja.API.csproj"

COPY . .

# run build over the API project
WORKDIR "/src/ContainerNinja.API/"
RUN dotnet build -c Release -o /app/build

# run publish over the API project
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

FROM base AS runtime
WORKDIR /app

COPY --from=publish /app/publish .
RUN ls -l
ENTRYPOINT [ "dotnet", "ContainerNinja.API.dll" ]
```

# DockerCompose Debug

```yml
version: '3.8'

networks:
  my-contacts:
    driver: bridge 

services:
  frontend:
    image: frontend:latest
    depends_on:
      - "api"
    build: 
      context: frontend/.
      dockerfile: debug.dockerfile
    volumes:
      - ./frontend:/app
    ports: 
      - "4200:4200"
    networks:
      - my-contacts 
  api:
    image: api:latest
    depends_on:
      - "postgres"
    build:
      context: Api/
      dockerfile: Debug.Dockerfile
    restart: always
    
    volumes:
      - ./Api:/app
      - ./Api:/src
    ports:
      - "8080:8000"  
    environment:
      DOTNET_USE_POLLING_FILE_WATCHER: 1
      ASPNETCORE_LOGGING__CONSOLE__DISABLECOLORS: "true"
      ASPNETCORE_ENVIRONMENT: "Development"
      ConnectionStrings:DefaultConnection: "Server=postgres;Database=contactdb;username=root;password=root"
      ConnectionStrings:IdentityConnection: "Server=postgres;database=contactdb;username=root;password=root"
      SendGridSettings:EmailFrom: "singhknitin@hotmail.com"
      SendGridSettings:Key: ""
      SendGridSettings:DisplayName: "Nitin Singh"

    networks:
      - my-contacts  
  postgres:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: contactdb
    volumes:
      - dbdata:/var/lib/postgresql/data
      # - ./dbscripts/seed. sql :/docker-entrypoint-initdb.d/seed.sql
    ports:
      - "5432:5432"
    networks:  
      - my-contacts  

  pgadmin:
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: root
    volumes:
      - dbadmin:/var/lib/postgresql/dbadmin
    ports:
      - "5050:80"
    networks:
      - my-contacts 
volumes:
  dbdata:
  dbadmin:
```

