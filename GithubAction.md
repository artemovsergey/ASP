# Github Action

```yml
name: CI/CD Pipeline

on:
  push:
    branches: [ master ]
jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_PASSWORD }}

      - name: Make script executable
        run: chmod +x ./build_and_push.sh

      - name: Run script to build and push images
        run: ./build_and_push.sh
      
      - name: Delete old repository from VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            rm -rf /home/artik3314/project
        
      - name: Clone or update repository from Github
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            git clone https://github.com/artemovsergey/React.git /home/artik3314/project/
            

    
      - name: Deploy docker-compose to VPS
        uses: appleboy/ssh-action@master
        with:
              host: ${{ secrets.VPS_HOST }}
              username: ${{ secrets.VPS_USERNAME }}
              key: ${{ secrets.SSH_PRIVATE_KEY }}
              script: |
                chmod +x /home/artik3314/project/vps.sh
                /home/artik3314/project/vps.sh
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
