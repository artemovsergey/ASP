# Теория
## Применение

- Контейнеризация Web приложений
- Построение отказоустойчивых систем
- Kubernetes
- Тестирование Web приложений
- CI/CD



## Установка docker на Ubuntu 22.04

### Хороший мануал. Прочитать
```
https://docs.docker.com/language/dotnet/develop/
```

### Переключение docker между Windows и Linux

Если полетел docker на WSL2 после запуска на Windows

В файле ```~/.docker/config.json``` будет "currentContext": "some-name"строчка. Вы можете удалить эту строку, чтобы вернуться к контексту по умолчанию. Если это последняя строка, обязательно удалите запятую в предыдущей строке, чтобы сохранить json действительным.



## Установка docker-compose в WSL2
```
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## Добавление пользователя в sudo группу в Ubuntu

```
usermod -aG sudo newuser
groups newuser
su - newuser
```



# Решение проблемы запуска docker в WSL

Установщик докеров использует iptables для nat. К сожалению, Debian использует nftables. Вы можете преобразовать записи в nftables или просто настроить Debian для использования устаревших iptables.

```sudo update-alternatives --set iptables /usr/sbin/iptables-legacy```

```sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy```

```dockerd``` должен нормально запускаться после перехода на ```iptables-legacy```.


# Просмотреть все активность сервисов

```sudo service --status-all```


# Запуск контейнера из образа

```sudo docker run -p 3001:80 aspapiimage54```

**Замечание**: -p 3001:80  => 3001 порт хоста, а 80 порт контейнера

# Dockerfile for ASP Core 7

```cmd
FROM mcr.microsoft.com/dotnet/aspnet:7.0
COPY publish_output .
ENTRYPOINT ["dotnet", "AspApiTest.dll"]c
```

# Dockercompose for ASP Core (ASP + Postgres + Adminer)

```cmd
version: '2'

services:

  testapp:
    image: aspapiimage54
    restart: always
    build:
      context: .
      dockerfile: .Dockerfile
    ports:
       - 3001:80

  db:
    image: postgres:13.0
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_PASSWORD: root
      POSTGRES_DB: postgres
       
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080



```

# Команды

### Собрать образ
```docker build --tag imagename .```

### Поднять образ
```docker run imagename```

### Собрать композицию образов
```docker-compose build``` 

### Поднять композицию образов c пересборкой
```docker-compose up -d --build```

## ASP Core + Postgres + Adminer
```
version: '2'

services:

  asp:
    build:
      context: .
      dockerfile: dockerfile
    ports:
       - 3000:80
    depends_on:
      - db

  db:
      image: postgres:13.0
      restart: always
      ports:
        - 5432:5432
      volumes:
        - postgres-data:/var/lib/postgresql/data

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
      
volumes:
  postgres-data:


```

## Строка подключения appsettings.json

```
 "PostgreSQL": "Host=db;Port=5432;Database=postgres;Username=postgres;Password=postgres",
```

---

## Dockerfile для WEB API ASP Core

```
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["WebApiTestLinux.csproj", "."]
RUN dotnet restore "WebApiTestLinux.csproj"
COPY . .
RUN dotnet build "WebApiTestLinux.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "WebApiTestLinux.csproj" -c Release -o /app/publish /p:UseAppHost=false
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApiTestLinux.dll"]
```

## Docker-compose for WEB API ASP Core

```yml
version: '3.4'

services:
  webapitestlinux:
    build:
      context: .
      dockerfile: WebApiTestLinux/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5000:80" 
```

