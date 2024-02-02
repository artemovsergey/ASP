# Поднимаем web-server под WSL

Основной стек: nginx + mysql + php-fpm.

Предполагается, что вы только что настроили WSL в своей системе и установили чистую ubuntu 18.04 LTS.

>
> Почему не apache? С точки зрения производительности, быстродействия, а как следствие - нагрузки - связка nginx + php-fpm показывает себя гораздо лучше. 
> Есть в этом решении и минусы - мы лишаемся удобного .htaccess и все необходимые настройки будем вынуждены производить в конфигах nginx и php,  но оно того стоит.
>

Статья в первую очередь ориентирована на так называемых `upper-junior`, хотя бы поверхностно знакомых с linux и используемым стеком.

## Установка необходимых пакетов

### Nginx

Устанавливаем и запускаем nginx

```bash
$ sudo apt update
$ sudo apt install nginx
$ sudo /etc/init.d/nginx start
```

Если вы все сделали верно, по ссылке http://127.0.0.1/ в браузере должна отобразиться стартовая страница "Welcome to nginx!".

На этом пока с nginx остановимся, вернемся уже на этапе конфигурации площадки.

### MySQL

```bash
$ sudo apt install mysql-server
$ sudo /etc/init.d/mysql start
```

Хоть у нас и локальная площадка, нужно сразу привыкать к правильному - запускаем штатный скрипт настройки политик безопасности mysql (установка паролей, требования к валидации, анонимный доступ и все такое):

```bash
$ sudo mysql_secure_installation
```

### PHP-fpm

```bash
$ sudo apt install php-fpm php-mysql
$ sudo /etc/init.d/php7.2-fpm start
```

## Настройка хоста

### Конфигурация nginx

Идем в папку с конфигами сайтов nginx и на базе  дефолтного конфига создаем свой. Файлы с конфигами лучше всего называть по домену, а для локальных проектов использовать какую-нибудь отдельную доменную зону, чтобы в дальнейшем не было путаницы - проект открыт локально или на удаленном сервере. Я обычно использую зону `.local` .

```bash
$ cd /etc/nginx/sites-available/
$ sudo cp default blog.local
```

Редактируем конфиг:

```bash
$ sudo nano blog.local
```
```nginx
server {
    listen 80;
    root /mnt/f/my/web/blog/public;
    index index.php index.html index.htm index.nginx-debian.html;
    server_name blog.local;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_buffering off;
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Так как у нас все-таки WSL 1 - будем использовать файловую системы основной системы. Логические диски внутрь WSL монтируются в `/mnt` - узнать список разделов можно командой `ls -la`.

Если хотите видеть свой сайт по тому же http://127.0.0.1/ или http://localhost/ - оставьте значения `server_name` и `listen` по умолчанию. В таком случае не получится завести несколько разных площадок, но и настраивать `hosts` не придется.

Чтобы сайт заработал, его конфиг нужно разместить в папке `/etc/nginx/sites-enabled/`, сделаем [симлинк](https://ru.wikipedia.org/wiki/Символическая_ссылка):

```bash
$ sudo ln -s /etc/nginx/sites-available/blog.local /etc/nginx/sites-enabled/
```

Заодно выключаем сайт по умолчанию, он нам больше не нужен:


```bash
$ sudo unlink /etc/nginx/sites-enabled/default
```

Тестируем конфиг (должно быть ok) и обновляем nginx:

```bash
$ sudo nginx -t
$ sudo /etc/init.d/nginx reload
```

### Hosts

Чтобы браузер понимал наш домен blog.local - он должен откуда-то получить ip-адрес сервера, связанного с этим доменом. В большом интернете этим занимаются DNS-сервера, а нам достаточно прописать соответствие в `hosts (C:\Windows\System32\drivers\etc\hosts)`. 

Добавляем следующую строку (нужны права администратора):

```
127.0.0.1		blog.local
```

### PHP

Если папка под проект к этому моменту еще не существует - стоит этим заняться.

Создаем `index.php`:

```bash
$ echo '<?php echo "Hello world!";' > /mnt/f/my/web/blog/public/index.php
```

Теперь открываем наш сайт в браузере - http://blog.local/. В случае успеха мы увидим текст `Hello world!`.

Также у нас есть возможность просмотреть все настройки сервера, [phpinfo](https://www.php.net/manual/ru/function.phpinfo.php) предоставляет нам такую возможность:


```bash
$ echo '<?php phpinfo();' > /mnt/f/my/web/blog/public/phpinfo.php
```

Открываем http://blog.local/phpinfo.php и видим свои настройки. 

**Важно!** Не используйте данную возможность на боевых серверах и тем более не оставляйте подобный файл лежать с доступом для всех. В выводе есть версии всех используемых дистрибутивов, это более чем достаточная информация если не для взлома сервера, так для направленной атаки, использующей уязвимости конкретных версий.



### MySQL
Проще всего, особенно для каки-то самописных проектов поставить админку. Для тех, кому это кажется лишним - можно создать пользователя для проекта и БД напрямую в консоли mysql.

Устанавливаем phpmyadmin вместе с необходимыми расширениями php и рестартим php-fpm, чтобы расширения включились.

```bash
$ sudo apt install phpmyadmin php-mbstring php-gettext
$ sudo /etc/init.d/php7.2-fpm restart
```
На вопрос "web-server to reconfigure automatically" не выбираем ничего, жмем Tab и Ok.
В окне "Configuring phpmyadmin" выбираем Yes и создаем пользователя для утилиты.

Теперь нам нужно настроить nginx, создаем еще один конфиг для хоста phpmyadmin:

```bash
$ cd /etc/nginx/sites-available/
$ sudo cp default phpmyadmin
$ sudo nano phpmyadmin
```

Содержимое (админка будет доступна по http://127.0.0.1/pma):

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html/;

    index index.php index.html index.htm index.nginx-debian.html;
    server_name _;

    location /pma {
        alias /usr/share/phpmyadmin/;
        location ~ \.php$ {
            fastcgi_buffering off;
            fastcgi_pass unix:/run/php/php7.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            include fastcgi_params;
            fastcgi_ignore_client_abort off;
        }
        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
            access_log    off;
            log_not_found    off;
            expires 1M;
        }
    }
}
```

Включаем сайт, проверяем конфиг и релоадим nginx:

```bash
$ sudo ln -s /etc/nginx/sites-available/phpmyadmin /etc/nginx/sites-enabled/
$ sudo nginx -t
$ sudo /etc/init.d/nginx reload
```

Открываем http://127.0.0.1/pma, пользователь `phpmyadmin`, пароль тот, что вводили при установке.

Если вдруг пользователь phpmysql оказался ограничен в правах (не может создать БД или других пользователей) - эти права всегда можно выдать вручную.

Идем в консоль mysql:

```bash
$ mysql -u root -p
```

И выдаем нужные права (в данном случае все возможные):

```sql
GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost' WITH GRANT OPTION;
```

Идем обратно в phpmyadmin и перезаходим в учетку, права должны быть на месте. Для демонстрации корректной работы всех пакетов добавим пользователя `blog` и БД `blog` с таблицей `test` произвольного формата (кодировку следует выбирать `utf8_general_ci`).

## Автозагрузка сервисов

Чтобы наши сервисы запускались автоматически при старте ubuntu - запускаем следующие команды:

```bash
$ sudo update-rc.d nginx defaults
$ sudo update-rc.d php7.2-fpm defaults
$ sudo update-rc.d mysql defaults
```

## Проверяем корректность работы

Подключаемся к БД из php и выводим на экран содержимое таблицы test (думаю, не стоит лишний раз упоминать, что доступы в открытом виде в коде указывать нельзя, для этого умные люди придумали .env файлы):

```php+HTML
<?php
try {
    $dbh = new PDO('mysql:host=localhost;dbname=blog', 'blog', 'blog');
    foreach($dbh->query('SELECT * from test') as $row) {
        echo "<pre>";
        print_r($row);
        echo "</pre>";
    }
    $dbh = null;
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

```

Открываем еще раз наш http://blog.local и видим содержимое таблицы на экране. 

В итоге у нас все работает, причем у нас в windows (!) есть привычная командная строка linux с привычными пакетами, которые используются в продакшне. То, для чего раньше ставились чрезвычайно медленные виртуалки, теперь доступно из коробки и заводится даже без танцев с бубнами. В удивительное время живем!

## Если что-то пошло не так

### ERR_NAME_NOT_RESOLVED (не удается получить доступ к сайту)

Браузер не смог найти указанный домен. Возможные причины:

- вы неправильно настроили конфиг сайта;
- вы не прописали в hosts домен или прописали его неверно;
- вы не сделали `nginx reload` после создания/изменения конфига сайта - это действие обязательно нужно выполнять после любых правок в конфиге. 

### 502 Bad Gateaway

Ошибка связана с тем, что nginx не получает ответа от бэкенда, в данном случае от php-fpm. 

Возможные причины:

- php-fpm не запущен;
- неверно указаны параметры php в настройках хоста nginx;
- ошибка непосредственно в php-скрипте.

Лучшие ваши помощники в устранении любой неполадки - логи:

```
/var/log/nginx/access.log  - лог запросов к nginx
/var/log/nginx/error.log   - лог ошибок nginx
/var/log/php7.2-fpm.log    - лог php-fpm
```

### Очень долгая загрузка страниц   (ERR_INCOMPLETE_CHUNKED_ENCODING)

Проверьте наличие директивы `fastcgi_buffering off;` в секции `location ~ \.php$` конфига сайта.

### upstream sent too big header while reading response header from upstream

Поднимите размер буфера в секции php

```
fastcgi_buffers 16 16k; 
fastcgi_buffer_size 32k;
```