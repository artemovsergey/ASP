# Настройка для SQLServer localdb

1. Тип подключения SQL Server (Old driver, jTDS)
2. Скачать драйвер https://github.com/milesibastos/jTDS/releases
3. В настройках выбрать новый драйвер
5. Подключение 
   sqllocaldb info
   sqllocaldb info <name>
   sqllocaldb start <name>
   Подключение: ./dbname;instance=LOCALDB#
7. Свойства NAMEDPIPE true
