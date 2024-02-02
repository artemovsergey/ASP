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

docker compose up --build
docker compose up --build -d
docker compose down

git stash -u  сохраняет все изменения перед созданием новой ветки

docker container ls  - список все запущенных контейнеров

docker exec -it 39fdcf0aff7b bash  - запуск команды в контейнере по id контейнера

example=# INSERT INTO "Students" ("ID", "LastName", "FirstMidName", "EnrollmentDate") VALUES (DEFAULT, 'Whale', 'Moby', '2013-03-20');

docker compose rm
docker compose up --build

docker compose watch

Запуск тестов
docker compose run --build --rm server dotnet test /source/tests

docker build -t dotnet-docker-image-test --progress=plain --no-cache --target build .

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
