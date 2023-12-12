## Кодировки и настройки
```SQL
show variables like "character_set_%";
```

## Резервное копирование базы данных

```sql
mysqldump -u username -p password -h localhost databasename > "C:\script.sql"
```
## Восстановление базы данных

```sql
mysql -u username -p password -h localhost databasename < "C:\script.sql"
```


## Проблемы

### Отображение русских символов через командную строку в MySQL
#### Решение:   в командной строке

 ```sql 
 set names "cp866" 
 ```
