# Github Action


## Unit и Integration Tests, Coverage and deploy results on Github Pages 

```yml
name: unit and integration Tests

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:latest
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: 9.0.102

      - name: Restore dependencies
        run: dotnet restore

      - name: Install reportgenerator tool
        run: dotnet tool install -g dotnet-reportgenerator-globaltool

      - name: Install coverlet.msbuild
        run: |
          # Находим все тестовые проекты и добавляем coverlet.msbuild
          for PROJECT in $(find . -name "*.Tests.csproj"); do
            dotnet add $PROJECT package coverlet.msbuild --version 3.2.0
          done
        continue-on-error: true  # На случай, если не найдены тестовые проекты

      - name: Build
        run: dotnet build --no-restore --configuration Release

      - name: Test with coverage
        env:
          POSTGRES_HOST: localhost
          POSTGRES_PORT: 5432
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        run: |
          dotnet test --no-build --configuration Release \
                --collect:"XPlat Code Coverage" \
                -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura \
                --logger trx

      - name: Generate report
        run: |
          reportgenerator -reports:"**/coverage.cobertura.xml" -targetdir:"coveragereport" -reporttypes:Html   

      - name: Publish test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            ./**/*.trx
            ./**/coverage.cobertura.xml

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./coveragereport
```

## Развертывание на VPS
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
cd /home/user1/tictactoe/

echo "Stop all containers"
docker-compose down

echo "Update new version images"
docker-compose pull

echo "Up"
docker-compose up -d --build

echo "Status"
docker ps
```
