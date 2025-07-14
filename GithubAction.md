# Github Action

- Развертывание на VPS
**Замечание**: в настройках репозитория на github надо укащать секреты. VPS_PRIVATE_KEY берет на локальном компьютере в папке `.ssh` c названием `id_rsa`, f `id_rsa.pub` должен соответствовать открытому публичному ключу на VPS 

```yml
name: deploy

on:
  push:
    branches: [ master ]
env:
  VPS_HOST: ${{ secrets.VPS_HOST }}
  VPS_USERNAME: ${{ secrets.VPS_USERNAME }}
  SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}

jobs:
  
  publish:

    runs-on: ubuntu-latest

    steps:
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '9.0'
          
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Delete old repository from VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            rm -rf /home/user1/tictactoe

      - name: Сlone new repo
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            git clone https://github.com/artemovsergey/TicTacToe.git /home/user1/tictactoe/

      - name: Deploy to VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            chmod +x /home/user1/tictactoe/scripts/vps.sh
            /home/user1/tictactoe/scripts/vps.sh
```


# build_and_push.sh

```bash
#!/bin/bash
echo "Build images from compose"
docker-compose build --no-cache --pull

echo "Login to dockerhub: "
docker login -u artik3314 -p Aa003314+ docker.io

echo "Images"
docker images

echo "Push service Nginx: "
docker tag react_nginx artik3314/react_nginx:latest
docker push artik3314/react_nginx:latest

echo "Push service Angular: "
docker tag react_angular artik3314/react_angular:latest
docker push artik3314/react_angular:latest

echo "Push service API: "
docker tag react_api artik3314/react_api:latest
docker push artik3314/react_api:latest
```


# vps.sh

```bash
#!/bin/bash
echo "Go to project"
cd /home/artik3314/project/

echo "Stop all containers"
docker-compose -f docker-compose-production.yml down

echo "Update new version images"
docker-compose -f docker-compose-production.yml pull

echo "Поднятие контейнеров"
docker-compose -f docker-compose-production.yml up -d --build

docker ps
```
