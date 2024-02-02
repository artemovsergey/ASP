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

