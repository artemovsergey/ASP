# Справка по командам

## Чтобы посмотреть в MySQL все текущие кодировки и настройки
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


### MySQL connector не работает с Visual Studio 2022 на начало 2022 года
#### Решение: подключение через NuGet пакет MySQL.Data в каждый проект или подключать локально

<!-- <details> 
  <summary> Problems </summary>
   A1: JavaScript 
</details> -->
****
