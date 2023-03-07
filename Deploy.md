# Развертывание приложения ASP Core на Ubuntu 22.04

1. В Visual Studio опубликоватьь проект ASP в ```Release``` (учитывать версию сервера Ubuntu и dotnet sdk)
2. Установить в ```Ubuntu 22.04``` в WSL
3. Перед установкой .NET выполните следующие команды, чтобы добавить ключ подписи пакета Microsoft в список доверенных ключей и добавить репозиторий пакетов.
```cmd
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```
4. Установите SDK
```cmd
sudo apt update && \
  sudo apt install -y dotnet-sdk-7.0
```
5. Возможно, нужна библиотека
```cmd
sudo apt install apt-transport-https

sudo apt update && \
  sudo apt install -y aspnetcore-runtime-7.0
```
**Замечание**: если будет ошибка, то поменять ```aspnetcore-runtime``` на ```dotnet-runtime```

6. Можно протестить без базы данных и веб-сервера ```nginx```, создать в домашней директории папку ```app``` в ней создать 

```dotnet new mvc```

```dotnet run```

7. Установка базы данных ```SQL Server``` (не работает в Ubuntu 22.04).

**Замечание**. Ubuntu 22.04 не работает пока с SQL Server 2019

Для Ubuntu 20.04:

На wsl устанавливается через настройку ручного запуска сервиса.

- Импортируйте открытые ключи GPG из репозитория:

```wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -```

- Зарегистрируйте репозиторий Ubuntu для SQL Server:

```sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/20.04/mssql-server-2022.list)"```

```sudo apt update```

```sudo apt install -y mssql-server```

- Установка MSSQL

```sudo /opt/mssql/bin/mssql-conf setup```

https://learn.microsoft.com/ru-ru/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver16

- Ручной запуск службы на WSL

```
sudo -u mssql /opt/mssql/bin/sqlservr -c \
    -d/var/opt/mssql/data/master.mdf \
    -l/var/opt/mssql/data/mastlog.ldf \
    -e/var/opt/mssql/log/errorlog \
    -x
```

8. Установка ```PostgreSQL``` на Ubuntu 20.04

```sudo apt update```

```sudo apt install postgresql postgresql-contrib```

```sudo -i -u postgres```

Запуск сервиса ```postgresql```

```sudo service postgresql start```

```psql```

https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart-ru

---

# Установка MySQL

```dotnet add package MySql.EntityFrameworkCore --version 7.0.0```

```Csharp
// Подключение базы данных MySQL 
string connection = builder.Configuration.GetConnectionString("MySQL");
builder.Services.AddDbContext<DataContext>(options => options.UseMySQL(connection));
```
```dotnet-ef migrations add Initial```

```dotnet-ef database update```

```sudo apt update```

```sudo apt install mysql-server```

```sudo service mysql start```

Конфигурация сервера ```MySQl```

```sudo mysql```
```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';```

https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04

Автоматические миграции

```Csharp
        public ProductController(DataContext context)
        {
            _context = context;
            _context.Database.Migrate();
        }
```

# Установка git

```sudo apt update```
```sudo apt install git```


# Настройка применения миграций при публикации приложения

https://stackoverflow.com/questions/37562122/is-there-a-way-to-run-ef-core-rc2-tools-from-published-dll/59269689#59269689

Простой способ 

```Csharp
context.Database.Migrate();
```
Права при публикации

```sudo chmod 777 -R AspMySqlTest```
