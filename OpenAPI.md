# Тестирвоание API c помощью http файла

```http
### localhost
@api = http://localhost:5055/api
### dockerhost
@api2 = http://localhost:8080/api


### Показать все игры 

GET {{api}}/games
Accept: application/json


### Получить отдельную игру

@gameId = b8aa36a5-9094-4dbd-80a5-444fcb92d3a4

GET {{api}}/games/{{gameId}}
Accept: application/json

### Создать игру

POST {{api2}}/games/new
Accept: application/json

### Сделать ход X или O

POST {{api2}}/games/{{gameId}}/move
Content-Type: application/json

{
  "x": 0,
  "y": 2,
  "p": "X"
}
```
