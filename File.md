# Отправка файлов

- Загрузка файла по пути. PhysicalFileResult

```Csharp
        public IActionResult GetFile()
        {

            // Файл должен быть в Debug 

            // Путь к файлу
            string file_path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "files/test.txt");
            // Тип файла - content-type
            string file_type = "text/plain";
            // Имя файла - необязательно
            string file_name = "test.txt";
            return PhysicalFile(file_path, file_type, file_name);
        }
```

- Отправка массива байтов

```Csharp
        // Отправка массива байтов
        public IActionResult GetBytes()
        {
            string path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "files/test.txt");
            byte[] mas = System.IO.File.ReadAllBytes(path);
            string file_type = "text/plain";
            string file_name = "hello2.txt";
            return File(mas, file_type, file_name);
        }
```

- Отправка потока

```Csharp
        // Отправка потока
        public IActionResult GetStream()
        {
            string path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "files/test.txt");
            FileStream fs = new FileStream(path, FileMode.Open);
            string file_type = "text/plain";
            string file_name = "hello3.txt";
            return File(fs, file_type, file_name);
        }
```

- VirtualFileResult

VirtualFileResult работает похожим образом, только возвращает файл по виртуальному пути. Здесь надо учитывать, что по умолчанию все пути к файлам в данном случае будут сопоставляться с папкой wwwroot. То есть нам надо помещать папки с файлами или отдельные файлы в каталог wwwroot:

```Csharp
public IActionResult GetVirtualFile() => File("files/test.txt", "text/plain", "hello4.txt");
```

Замечание: Во всех выше перечисленных случаях использование имени файла в качестве третьего параметра метода File/PhysicalFile необязательно. А вот тип файла обязательно надо передавать. Но подобное поведение может быть не всегда удобным: мы можем точно не знать тип отправляемых файлов, или файлы представляют самые разные типы. И в этом случае мы можем использовать универсальный тип application/octet-stream.

Примечание:Передача через массив байтов выручает, когда данные хранятся... в массиве байтов.

Например:
В таблице базы данных могут быть поля, в которых хранятся картинки, видео и т.д.
Вот эти поля(если я не ошибаюсь) имеют тип BINARY или VARBINARY.
В C# их мы загружаем в byte[], затем передаем куда-то там...

Передача через поток данных(stream) выручает тогда, когда мы не хотим занимать лишнюю память при передаче.
Вот в примере сверху мы сначала заняли память под массив byte[], а потом отправили его.
А открыв поток, мы как бы одновременно считываем и передаем, без промежуточного хранения.
