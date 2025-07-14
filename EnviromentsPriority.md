1. builder.WebHost.UseUrls("http://*:1234");


2.

```yaml

api:
  container_name: TicTacToeAPI
  image: tictactoe.api
  environment:
    - ASPNETCORE_ENVIRONMENT = Production
    - ASPNETCORE_URLS=http://*:5000
```
docker-compose переопределяет переменные окружения в Dockerfile

3.

```json

{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://*:5055"
      }
    }
  }
}

```

4. Порт по умолчанию: 8080
