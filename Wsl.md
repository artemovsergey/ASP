wsl --list --online
wsl --install -d Ubuntu-20.04

1. WSL образ не запускает установленные пакеты после установки. Например, после установки mysql вам нужно запустить этот пакет командой sudo service mysql start. И так с любым другим пакетом.

2. WSL не запускает установленные пакеты при запуске системы.. После перезагрузки компьютера и старта WSL образа вам нужно запустить все необходимые для вас пакеты. Чтобы не писать каждый раз все эти команды, проще в корневой папке (папке ~), откуда стартует ваш образ, создать файл start.sh:

```
#!/bin/bash
service nginx start
service mysql start
service php8.0-fpm start
service php8.1-fpm start
service php7.4-fpm start
service redis-server start
service cron start
```

Далее сохраняем этот файл. Теперь после старта WSL для того, чтобы выполнить все команды из этого файла, нам нужно выполнить команду:

sudo sh start.sh

прописать нужный нам домен (домены) в файле C:\Windows\System32\drivers\etc\hosts (не забывайте открывать от имени Администратора) следующим образом:

127.0.0.1    ilavista.test
::1          ilavista.test

netstat -tulnp | grep httpd



