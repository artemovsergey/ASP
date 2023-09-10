# ASP Core

https://learn.microsoft.com/ru-ru/aspnet/core/?view=aspnetcore-7.0

## Паттерны проектирования

![](/Patterns.png)

# dotnet cli

```
dotnet new sln
dotnet new webapi -o APIname
dotnet sln add APIname

dotnet new gitignore

```

# Настройка VS Code для .NET

- FontSize
- Material Icon
- Exclude **/bin и **/obj
- Расширение для C#

# Теория
**Интерфейс** определяет свойства и методы, предназначенные для доступа к данным, а для работы с механизмом хранения данных применяется **класс реализации**. Это паттерн ```Repository```

# Cors

```Csharp
builder.Services.AddCors();
app.UseCors(x => x.AllowAnyHeader().AllowAnyMethod().WithOrigins("http://localhost:4200"));
```

# NodeModules

```Csharp
app.UseNodeModules();
```

# Program.cs

```csharp

using SportStore.Models;
using SportStore.Interface;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.SqlServer;
using Microsoft.Extensions.FileProviders;
using Microsoft.Extensions.Options;
using System.Configuration;

var builder = WebApplication.CreateBuilder(args);


builder.Services.AddMvc();
//добавляет все сервисы фреймворка MVC
// (в том числе сервисы для работы с аутентификацией и авторизацией, валидацией и т.д.)

// AddMvcCore(): добавляет только основные сервисы фреймворка MVC,
// а всю дополнительную функциональность, типа аутентификацией и авторизацией, валидацией и т.д.,
// необходимо добавлять самостоятельно

// AddControllersWithViews(): добавляет только те сервисы фреймворка MVC,
// которые позволяют использовать контроллеры и представления и связанную функциональность.
// При создании проекта по типу ASP.NET Core Web App (Model-View-Controller) используется именно этот метод

// AddControllers(): позволяет использовать контроллеры, но без представлений.

//builder.Services.AddControllersWithViews();
//builder.Services.AddMvc(options => options.EnableEndpointRouting = false);

// Подключение базы данных SQL Server
string connection = builder.Configuration.GetConnectionString("PostgreSQL");
//builder.Services.AddDbContext<DataContext>(options => options.UseSqlServer(connection));
builder.Services.AddDbContext<DataContext>(options => options.UseNpgsql(connection));
//builder.Services.AddDbContext<DataContext>(options => options.UseMySql(connection, new MySqlServerVersion(new Version(8, 0, 30))));
//builder.Services.AddDbContext<DataContext>(options => options.UseSqlite(connection));

// Это значит совместное использоване объекта класса AppTimeService во всем приложении.
// Общая функциональность для приложения
builder.Services.AddSingleton<AppTimeService>();


//Какие преимущества? Нам не придется возиться с созданием объекта, удалением его или управлением этим объектом в коде наших страниц.


// Создание служю необходимых для управления сеансом
builder.Services.AddMemoryCache();
builder.Services.AddSession();

// Сервисы, это любой функционал, который мы хотим зарегистрировать, чтобы другие части приложения, могли его использовать (эл почта, бд…).

// Configure the HTTP request pipeline.


builder.Services.AddTransient<IRepository,DataRepository>(); // одие раз создается одиночный объект для всего приложенияы
builder.Services.AddTransient<ICategoryRepository, CategoryRepository>();
builder.Services.AddTransient<IOrderRepository, OrderRepository>();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
app.UseDeveloperExceptionPage(); // Ошибки при отладке
app.UseHttpsRedirection();
app.UseStaticFiles(); // доставка статического содержимого
app.UseStatusCodePages(); // отправка кодов состояния в ответе

// добавляет в конвейер новый компонент, который ассоциирует данные сеанса с запросами
// и добавляет cookie наборы к ответам
// должен вызываться перед методом app.UseMvc()
app.UseSession();


//app.UseMvc();
//app.UseMvcWithDefaultRoute();
//app.MapDefaultControllerRoute();

// Определение маршрутов
app.MapControllerRoute(
  name: "default",
  pattern: "/{controller=Home}/{action=Index}/{id?}");

// тоже самое
//app.MapDefaultControllerRoute();


app.Run();
//

public class AppTimeService { };
```

# appsettings.json

```JSON

{
  "ConnectionStrings": {



    "MSSQL": "Server=WIN-PO9SVP3KRMT\\MSSQLSERVER01;Database=SportStore;Trusted_Connection=True;MultipleActiveResultSets=true",
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SportStore;Trusted_Connection=True;MultipleActiveResultSets=true",
    "PostgreSQL": "Host=localhost;Port=5432;Database=SportStore;Username=postgres;Password=root",
    "MySQL": "server=localhost;user=root;password=root;database=SportStore;",
    "SQLite": "Data Source=SportStore.db"

  },
  "Logging": {
    "LogLevel": {
      "Default": "None",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "AllowedHosts": "*"
}

```

# Scaffold

**Замечание**: надо установить ```Microsoft.EntityFrameworkCore.Tools```.

В консоли диспетчера пакетов Nuget прописать команду

```Scaffold-DbContext "Server=localhost;Database=Users;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models```
	
Команда создает модели из каждой сущности в базе данных, учитывая связи, а также создает класс контекста для работы с данными как с классами.

```Scaffold-DbContext "Data Source=.\ComputerDatabase.db" Microsoft.EntityFrameworkCore.Sqlite -OutputDir Models```

**Примечание**: если Scaffold для SQLite, ему нужна база из проекта, а не в ```Debug```. При инициализации контекста база данных ```SQLite``` создается в ```Debug``` по умолчанию.


В консоле диспетчера пакетов для SQLServer

```
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=UserDatabase;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models2
```

# Команды

```dotnet new mvc --output TestMVC```

```dotnet tool install -g dotnet-aspnet-codegenerator```

```dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design```

```dotnet add package Microsoft.EntityFrameworkCore.Design```

```dotnet tool install --global dotnet-ef```

```dotnet add package Microsoft.EntityFrameworkCore.SQLite```

```dotnet add package Microsoft.EntityFrameworkCore.SqlServer```

```dotnet aspnet-codegenerator controller --controllerName Home --model Beer --dataContext BeerContext --useDefaultLayout -outDir Controllers -f --useSqlite```

https://learn.microsoft.com/ru-ru/aspnet/core/fundamentals/tools/dotnet-aspnet-codegenerator?view=aspnetcore-7.0


```dotnet ef database drop --force```

```dotnet ef migrations add InitialCreate```

**Замечание:** возможно надо перезапустить Visual Studio

**Замечание**: --project Name, если несколько проектов. Помогает также очистка и пересборка решения.
```dotnet ef database update```

**Замечание**: --connection "Data Source=My.db"

Установка доверия к сертификату разработки

```csharp
dotnet dev-certs https --trust
```   



# Context for SQLite
```Csharp
 public class UserContext : DbContext
    {

        public DbSet<User> Users { get; set; }

        public UserContext(DbContextOptions<UserContext> options): base(options)
        {
            Database.EnsureCreated();
        }


        //protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        //{
        //    //Don't hardcode like this in a real app:
        //    //optionsBuilder.UseSqlite("Data Source=beers.db");
        //}

    }
```

# Файл _ViewStart

```Csharp
@{
    Layout = "_Layout";
}
```
# Файл _ViewImports

```Csharp
@using AppTest.Models;
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

# Метод GET для вывода всех пользователей (Index)
```csharp

// Запрос на вывод все пользователей на /Home/index

[HttpGet]
public IActionResult Index()
{
    return View(db.Users.ToList());
}

```

# Метод GЕT на вывод одного пользователя (Show/id)

```csharp

// запрос на одного пользователя /Home/SelectUser/id

[HttpGet]
public IActionResult SelectUser(int? id)  
{
    if (id == null) return RedirectToAction("Index");

    ViewBag.UserId = id;
    var selectUser = db.Users.Include(u => u.Products).FirstOrDefault(u => u.Id == id);


    // возврат на одноименное представление контроллера
    return View(selectUser);
}

```

# Метод POST на добавление пользователя (create)

```csharp

        [HttpPost]
        //  Home/AddUser
        public IActionResult AddUser(User user) // форма по аттрибутам поймет, что передается user
        {
            db.Users.Add(user);
            // сохраняем в бд все изменения
            try
            {
                db.SaveChanges();
            }
            catch(Exception ex)
            {
                ViewBag.Error = ex.Message.ToString();
            }


            return RedirectToAction("Index");

        }

```
# Представление на добавление пользователя

```cshtml
@model User;

<div>
<h2>Форма добавления пользователя</h2>

<form asp-action="Index"   method="post"  role="form" >
  
  <div asp-validation-summary = "All" ></div>


  <input type="hidden"  asp-for="Id" value="1" />

  <div class="mb-3">
    <label for="exampleInputEmail1" class="form-label">Имя</label>
    <input type="text" asp-for="Name"    class="form-control" aria-describedby="emailHelp">
    <div id="textHelp" class="form-text">Имя пользователя</div>
  </div>

  <div class="mb-3">
    <label for="exampleInputEmail1" class="form-label">Возраст</label>
    <input type="text" asp-for="Age" class="form-control" aria-describedby="emailHelp">
    <div id="textHelp" class="form-text">Возраст</div>
  </div>
  
  <button type="submit" class="btn btn-primary">Сохранить</button>
</form>

<br>

<a href="/Home/Users" > Список пользователей</a>

</div>
```
# Представление на редактирование пользователя

```cshtml

@model User;


@{
    ViewData["Title"] = "Добавление пользователя";
}

@{
    Layout ="_Layout"; 
}

<h2>Форма добавления пользователя</h2>
 
<form method="post" class="form-horizontal" role="form">
    
    
    <!--  <input type="hidden" value="@ViewBag.UserId" name="Id" /> -->
    

    <p>Для оформления покупки заполните следующие поля:</p>
    <!--
    <div class="form-group">
        <label for="User" class="col-md-2 control-label">Id:</label>
        <div class="col-md-4">
            <input type="text" name="Id" class="form-control" />
        </div>
    </div>
    -->
    <div class="form-group">
        <label for="Address" class="col-md-2 control-label">Имя:</label>
        <div class="col-md-4">
            <input type="text" value = @Model.Name  name="Name" class="form-control" />
        </div>
    </div>


    <div class="form-group">
        <label class="col-md-2 control-label">Возраст:</label>
        <div class="col-md-4">
            <input type="text" value = @Model.Age  name="Age" class="form-control" />
        </div>
    </div>

    <div class="form-group">
        <div class="col-md-offset-2 col-md-10">
            <input type="submit" class="btn btn-default" value="Отправить" />
        </div>
    </div>
</form>




```
# Метод GET на редактирование пользователя (edit/id)

```csharp

[HttpGet]
// Home/AddUser
public IActionResult EditUser(int id) // просто отображаем форму
{
    User user = db.Users.Where(u => u.Id == id).FirstOrDefault();


    return View(user);
}

```
# Метод POST на редактирование пользователя (edit)

```csharp

 [HttpPost]
 // Home/AddUser
 public IActionResult EditUser(User user) // просто отображаем форму
 {
     db.Users.Update(user);
     // сохраняем в бд все изменения
     try
     {
         db.SaveChanges();
     }
     catch (Exception ex)
     {
         ViewBag.Error = ex.Message.ToString();
     }


     return RedirectToAction("Index");
 }

```
# Метод DELETE на удаление пользователя

```csharp

[HttpGet]
// Home/DeleteUser
public IActionResult DeleteUser(int id) // просто отображаем форму
{
    User user = db.Users.Where(u => u.Id == id).FirstOrDefault();
    db.Users.Remove(user);

    try
    {
        db.SaveChanges();
    }
    catch (Exception ex)
    {
        ViewBag.Error = ex.Message.ToString();
    }


    return RedirectToAction("Index");
}

```

# Index.cshtml

```cshtml

@model IEnumerable<FurnitureShop.Models.User>

    <!-- это модель представления, т.е. объект(объекты) какой модели контроллер передает в представление  -->
    <!-- в данном случае модель представления совпадает с просто моделью -->


@{
    // В даннос случае макет для данного представления отключен
    //Layout = null;



    // В даннос случае макет для данного представления включен
    Layout = "_Layout";
}

    <!--  @Model - это объект-список сейчас -->


<h1>    
     Мебельный магазин
     <!-- _GetMessage это частичное представление. Оно должно находиться в папке View для своего контроллера -->
        
</h1>    


<table class="table">
  <thead>
    <tr>
      <td scope="col">#</td>
      <td scope="col">Name</td>
      <td scope="col">Age</td>
      <td scope="col">#</td>
      <td scope="col">#</td>
    </tr>
  </thead>
  <tbody>
          @foreach (var user in Model)
                {
                    <tr>
                        <td>@user.Name</td>
                        <td>@user.Age</td>
                        <td><a href="~/Home/SelectUser/@user.Id">Просмотр</a></td>
                        <td><a href="~/Home/EditUser/@user.Id">Редактировать</a></td>
                        <td><a href="~/Home/DeleteUser/@user.Id">Удалить</a></td>         
                    </tr>
                }
  </tbody>
</table>


<a class = "btn btn-primary" href="~/Home/AddUser">Добавить пользователя</a>

```

# Пример макета или мастер-страницы _Layout.cshtml

```cshtml

<!DOCTYPE html>
<html>
    
    
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>@ViewData["Title"]</title>
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </head>


    <body>
    
        <!--меню-->
    <nav class="navbar navbar-expand-lg bg-light">
      <div class="container-fluid">
        <a class="navbar-brand" href="/Product/">Мебельный магазин</a>
        
          <form class="d-flex" role="search">
            <input class="form-control me-2" type="search" placeholder="Найдите мебель" aria-label="Search">
            <button class="btn btn-outline-success" type="submit">Поиск</button>
          </form>

          <a class="navbar-brand" href="#">Корзина</a>
        
      </div>
    </nav>

        <!--основной контент-->
        <div class="container body-content">
            
            @RenderBody()
            
            <hr />



            <footer>
                <p>© 2022 - TestStore</p>
            </footer>
        </div>


    </body>
</html>

```
 
**Замечание**: можно создать отдельный файл класса с глобальными подключениями ```global using Microsoft.AspNetCore.Mvc ```


# Request. Получение данных из контекста запроса

Параметры представляют самый простой способ получения данных, но в действительности нам необязательно их использовать. В контроллере доступен объект Request, у которого можно получить как данные строки запроса, так и данные отправленных форм.

Данные строки запроса доступны через свойство Request.Query, которое представляет объект IQueryCollection. Например:

```csharp
public string Area()
{
    string altitudeString = Request.Query.FirstOrDefault(p => p.Key == "altitude").Value;
    int altitude = Int32.Parse(altitudeString);
 
    string heightString = Request.Query.FirstOrDefault(p => p.Key == "height").Value;
    int height = Int32.Parse(heightString);
 
    double square = altitude * height / 2;
    return $"Площадь треугольника с основанием {altitude} и высотой {height} равна {square}";
}
```

В данном случае метод Area обрабатывает GET-запросы, и мы можем к нему обратиться через запрос типа http://localhost:57086/Home/Area?altitude=20&height=4.

# Request. Получение данных отправленных форм

Для получения данных отправленных форм можно использовать свойство ```Request.Form```. Это свойство представляет объект IFormsCollection, но работает аналогично Request.Query:

```csharp
[HttpPost]
public string Area()
{
    string altitudeString = Request.Form.FirstOrDefault(p => p.Key == "altitude").Value;
    int altitude = Int32.Parse(altitudeString);
 
    string heightString = Request.Form.FirstOrDefault(p =>p.Key == "height").Value;
    int height = Int32.Parse(heightString);
 
    double square = altitude * height / 2;
    return $"Площадь треугольника с основанием {altitude} и высотой {height} равна {square}";
}
```
# Переадресация

### Теория

Протокол HTTP поддерживает два типа переадресации:

- постоянная переадресация. При постоянной переадресации сервер будет отправлять браузеру статусный код 301. При данном типе переадресации предполагается, что запрашиваемый документ окончательно перемещен в другое место. И после получения статусного кода 301 браузер может автоматически настраивать запросы на новый ресурс, даже если старый ресурс со временем перестанет применять переадресацию. Поэтому данный способ можно использовать, если вы полностью уверены, что документ на старое место уже не возвратится..

- временная переадресация. При временной переадресации сервер будет отправлять браузеру статусный код 302. При этом считается, что запрашиваемый документ временно перемещен на другую страницу.

В обоих случаях для создания переадресации может использоваться объект ```RedirectResult```, однако метод, возвращающий данный объект, будет отличаться.


Для временной переадресации применяется метод Redirect
```return Redirect("~/Home/About");```

Для создания переадресации на определенный метод контроллера используется объект RedirectToActionResult. 
```return RedirectToAction("About", "Home", new { name = "Tom", age = 37 });```

Для переадресации на имя маршрута RedirectToRouteResult
```return RedirectToRoute("default", new { controller = "Home", action = "About", name = "Tom", age = 22 });```

```csharp
// Переадресация
    public IActionResult TestRedirect(bool about)
    {
        
        string a = "http://www.stvcc.ru";
        if (about == true)
        {
            return Redirect("~/Home/About");
        }
        else
        {
            return Redirect(a);
        }
    }
```

# Отправка статусных кодов

Нередко возникает необходимость отправить в ответ на запрос какой-либо статусный код. Например, если пользователь пытается получить доступ к ресурсу, который недоступен, или для которого у пользователя нету прав. Либо если просто нужно уведомить браузер пользователя с помощью статусного кода об успешном выполнении операции, как иногда применяется в ajax-запросах. Для этого в ASP.NET Core MVC определена богатая палитра классов, которые можно использовать для отправки статусного кода.

StatusCodeResult позволяет отправить любой статусный код клиенту:

```csharp
public IActionResult Index()
{
    return StatusCode(401);
}
```

Подобным образом мы можем послать браузеру любой другой статусный код. Но для отдельных кодов статуса предназначены свои отдельные классы.

NotFoundResult и NotFoundObjectResult посылает код 404, уведомляя браузер о том, что ресурс не найден. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.
Объекты обоих классов создаются методом NotFound. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию. Например, используем NotFoundObjectResult:

```return NotFound("Resource not found");```

UnauthorizedResult посылает код 401, уведомляя пользователя, что он не автризован для доступа к ресурсу:

```return Unauthorized();```

UnauthorizedObjectResult также посылает код 401, только позволяет добавить в ответ некоторый объект с информацией об ошибке:

```return Unauthorized(new Error("Access is denied"));```

BadResult и BadObjectResult посылают код 400, что говорит о том, что запрос некорректный. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.
Эти классы можно применять, например, если в запросе нет каких-то параметров или данные представляют совсем не те типы, которые мы ожидаем получить, и т.д.
Объекты обоих классов создаются методом BadRequest. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию:

```return BadRequest("Name undefined")```

OkResult и OkObjectResult посылают код 200, уведомляя об успешном выполнении запроса. Второй класс в дополнении к статусному коду позволяет отправить дополнительную информацию, которая потом отправляется клиенту в формате json.

Объекты обоих классов создаются методом Ok(). Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию:

```return Ok("Don't worry. Be happy");```

# Отправка файлов

Загрузка файла по пути. PhysicalFileResult

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
Отправка массива байтов

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

Отправка потока

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

VirtualFileResult

VirtualFileResult работает похожим образом, только возвращает файл по виртуальному пути. Здесь надо учитывать, что по умолчанию все пути к файлам в данном случае будут сопоставляться с папкой wwwroot. То есть нам надо помещать папки с файлами или отдельные файлы в каталог wwwroot:

```Csharp
public IActionResult GetVirtualFile() => File("files/test.txt", "text/plain", "hello4.txt");
```

Замечание: Во всех выше перечисленных случаях использование имени файла в качестве третьего параметра метода File/PhysicalFile необязательно. А вот тип файла обязательно надо передавать. Но подобное поведение может быть не всегда удобным: мы можем точно не знать тип отправляемых файлов, или файлы представляют самые разные типы. И в этом случае мы можем использовать универсальный тип application/octet-stream.

Примечание:Передача через массив байтов выручает, когда данные хранятся... в массиве байтов(логично да).

Например:
В таблице базы данных могут быть поля, в которых хранятся картинки, видео и т.д.
Вот эти поля(если я не ошибаюсь) имеют тип BINARY или VARBINARY.
В C# их мы загружаем в byte[], затем передаем куда-то там...

Передача через поток данных(stream) выручает тогда, когда мы не хотим занимать лишнюю память при передаче.
Вот в примере сверху мы сначала заняли память под массив byte[], а потом отправили его.
А открыв поток, мы как бы одновременно считываем и передаем, без промежуточного хранения.

# Переопределение контроллеров

Как правило, для создания контроллера достаточно унаследовать свой класс от базового класса Controller. Однако если нам необходимо, чтобы наши контроллеры реализовали некоторую общую логику, мы можем определить свой базовый класс контроллера и уже от него наследовать остальные контроллеры. Либо мы также можем переопределить некоторые методы базового класса Controller, если они нас не устраивают.

Что мы можем в контроллере переопределить? В базовом классе Controller среди всех прочих методов есть три интересных метода:

- Метод OnActionExecuting() выполняется при вызове метода контроллера до его непосредственного выполнения.

- Метод OnActionExecuted() выполняется после выполнения метода контроллера.

- Метод OnActionExecutionAsync() представляет асинхронную версию метода OnActionExecuting().

**Замечание**: это похоже на callback или фильтры перед действиями контроллеров в Rails


# Передача зависимостей в контроллер

Как и любой класс, контроллер может получать **сервисы приложения** через механизм dependency injection. В контроллере это можно делать следующими способами:

- Через конструктор
- Через параметр метода, к которому применяется атрибут **FromServices**
- Через свойство ```HttpContext.RequestServices```

Например, пусть у нас есть следующий файл Program.cs:

```Csharp
var builder = WebApplication.CreateBuilder(args);
 
builder.Services.AddControllers();  // добавляем поддержку контроллеров
builder.Services.AddTransient<ITimeService, SimpleTimeService>(); // добавляем сервис ITimeService
 
var app = builder.Build();
 
// устанавливаем сопоставление маршрутов с контроллерами
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
 
app.Run();
 
public interface ITimeService
{
    string Time { get; }
}

public class SimpleTimeService : ITimeService
{
    public string Time =>  DateTime.Now.ToShortTimeString();
}

```

В данном случае определен интерфейс ```ITimeService``` и его реализация ```SimpleTimeService```. И в приложении происходит регистрация сервиса ```ITimeService```.

**Передача через конструктор**

Когда приходит запрос к контроллеру, инфраструктура MVC вызывает провайдер сервисов для создания объекта HomeController. Провайдер сервисов проверят конструктор класса HomeController на наличие зависимостей. Затем создает объекты для всех используемых зависимостей и передает их в конструктор ```HomeController``` для создания объекта контроллера, который затем обрабатывает запрос.

Например, получим зависимость в конструкторе контроллера:

```Csharp
public class HomeController : Controller
{
    readonly ITimeService timeService;
 
    public HomeController(ITimeService timeServ)
    {
        timeService = timeServ;
    }
    public string Index() => timeService.Time;
}
```

В данном случае процесс установки зависимостей будет выглядеть следующим образом:

Приложение получает запрос к методу контроллера ```HomeController```. Фреймворк MVC обращается к провайдеру сервисов для создания объекта контроллера ```HomeController```
Провайдер сервисов смотрит на конструктор класса ```HomeController``` и видит, что там имеется зависимость от интерфейса ```ITimeService```
Провайдер сервисов среди зарегистрированных зависимостей ищет класс, который представляет реализацию интерфейса ```ITimeService```
Если нужная зависимость найдена, то провайдер сервисов создает объект класса, который реализует интерфейс ```ITimeService```
Затем провайдер сервисов создает объект ```HomeController```, передавая в его конструктор ранее созданную реализацию ```ITimeService```
В конце провайдер сервисов возвращает созданный объект ```HomeController``` инфраструктуре MVC, которая использует контроллер для обработки запроса

- Передача зависимостей в методы. FromServices

Иногда зависимость используется только в одном методе. И в этом случае нет необходимости передавать ее в контроллер, поскольку она напрямую может быть внедрена в сам метод, который ее использует. Для передачи зависимости в метод применяется атрибут [FromServices]:

```Csharp
public string Index([FromServices] ITimeService timeService)
{
    return timeService.Time;
}
```
Атрибут FromServices предоставляется инфраструктурой MVC - он определен в пространстве имен ```Microsoft.AspNetCore.Mvc.HttpContext.RequestServices```

В методах контроллера можно обращаться к объекту контекста запроса через свойство ```HttpContext```, а через свойство ```HttpContext.RequestServices``` можно получить все зарегистрированные в приложении сервисы:

```Csharp
public string Index()
{
    ITimeService? timeService = HttpContext.RequestServices.GetService<ITimeService>();
    return timeService?.Time ?? "Undefined";
}

```

# Представление Views

# Представления Razor

**Замечание**: полноценной html-страницей представление все равно не является, потому что во время выполнения эти представления компилируются в сборки и уже затем используются для генерации html-страниц, которые видит пользователь в своем браузере.

# Применение мастер-страницы

```csharp
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>@ViewData["Title"]</title>
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </head>
    <body>
        <!--меню-->
        <nav class="navbar navbar-inverse">
            <div class="container">
                <div>
                    <ul class="nav navbar-nav">
                        <li><a href="~/Home/Index" class="navbar-brand">Главная</a></li>
                    </ul>
                </div>
            </div>
        </nav>
        <!--основной контент-->
        <div class="container body-content">
            @RenderBody()
            <hr />
            <footer>
                <p>© 2022 - TestStore</p>
            </footer>
        </div>
    </body>
</html>

```


# Внедрение зависимостей

Dependency injection (DI) или внедрение зависимостей представляет механизм, который позволяет сделать взаимодействующие в приложении объекты слабосвязанными. Такие объекты связаны между собой через абстракции, например, через интерфейсы, что делает всю систему более гибкой, более адаптируемой и расширяемой.

В центре подобного механизма находится понятие зависимость - некоторая сущность, от которой зависит другая сущность. Например:

```Csharp
class Logger
{
    public void Log(string message) => Console.WriteLine(message);
}
class Message
{
    Logger logger = new Logger();
    public string Text { get; set; } = "";
    public void Print() => logger.Log(Text);
}
```

Здесь сущность Message, которая представляет некоторое сообщение, зависит от другой сущности - Logger, которая представляет логгер. В методе Print() класса Message имитируется логгирование текста сообщения путем вызова у объекта Logger метода Log, который выводит сообщение на консоль. Однако здесь класс Message тесно связан с классом Loger. Класс Message отвечает за создание объекта Logger. Это имеет ряд недостатков. Прежде всего, если мы захотим вместо класса Logger использовать другой тип тип логгера, например, логгировать в файл, а не на консоль, то нам придется менять класс Message. Один класс не составит труда поменять, но если в проекте таких классов много, то поменять во всех класс Logger на другой будет труднее. Кроме того, класс Logger может иметь свои зависимости, которые тоже может потребоваться поменять. В итоге такими системами сложнее управлять и сложнее тестировать.
Чтобы отвязать объект Logger от класса Message, мы можем создать абстракцию, которая будет представлять логгер, и передавать ее извне в объект Message:

```Csharp
interface ILogger
{
    void Log(string message);
}
class Logger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}
class Message
{
    ILogger logger;
    public string Text { get; set; } = "";
    public Message(ILogger logger)
    {
        this.logger = logger;
    }
    public void Print() => logger.Log(Text);
}
```

Теперь класс Message не зависит от конкретной реализации класса Logger - это может быть любая реализация интерфейса ILogger. Кроме того, создание объекта логгера выносится во внешний код. Класс Message больше ничего не знает о логгере кроме того, что у него есть метод Log, который позволяет логгировать его текст.
Тем не менее остается проблема управления подобными зависимостями, особенно если это касается больших приложений. Нередко для установки зависимостей в подобных системах используются специальные контейнеры - IoC-контейнеры

Такие контейнеры служат своего рода фабриками, которые устанавливают зависимости между абстракциями и конкретными объектами и, как правило, управляют созданием этих объектов.
Преимуществом ASP.NET Core в этом оношении является то, что фреймворк уже по умолчанию имеет встроенный контейнер внедрения зависимостей, который представлен интерфейсом IServiceProvider. А сами зависимости еще называются сервисами, собственно поэтому контейнер можно назвать провайдером сервисов. Этот контейнер отвечает за сопоставление зависимостей с конкретными типами и за внедрение зависимостей в различные объекты

**Примечание**:  Статические переменные класса - это способ хранить состояние объекта в памяти (во время работы приложения)

# Управление привязкой модели
https://metanit.com/sharp/aspnetmvc/5.3.php

# Внедрение зависимостей в представления

ASP.NET Core MVC поддерживает внедрение зависимостей в представления. За внедрение зависимостей в представление отвечает директива @inject.

Допустим, у нас в проекте в файле Program.cs внедряется сервис ITimeService:

```Csharp
var builder = WebApplication.CreateBuilder(args);
 
// добавляем поддержку контроллеров с представлениями
builder.Services.AddControllersWithViews();
// внедряем сервис ITimeService
builder.Services.AddTransient<ITimeService, SimpleTimeService>();
var app = builder.Build();
 
// устанавливаем сопоставление маршрутов с контроллерами
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
 
app.Run();
 
public interface ITimeService
{
    string Time { get; }
}
public class SimpleTimeService : ITimeService
{
    public string Time =>  DateTime.Now.ToShortTimeString();
}
```

Интерфейс ITimeService определяет свойство Time. В реализации этого интерфейса - классе SimpleTimeService данное свойство возвращает строку с текущим временем. И затем класс SimpleTimeService применяется в качестве реализации для сервиса ITimeService.

И далее мы можем использовать зависимости в представлении. Например, изменим какое-нибудь представление:

```Csharp
@inject ITimeService timeService
<h2>Current Time: @timeService.Time</h2>
```
Директива @inject принимает два параметра: первый параметр представляет тип сервиса (в данном случае ITimeService), а второй - название переменной этого типа (в данном случае timeService).

# Передача данных в представление

Существуют различные способы передачи данных из контроллера в представление:

- ViewData
- ViewBag
- Модель представления

**ViewData**

ViewData представляет словарь из пар ключ-значение:

```csharp
public IActionResult Index()
{
    ViewData["Message"] = "Hello ASP.NET Core";
 
    return View();
}
```

Здесь динамически определяется во ViewData объект с ключом "Message" и значением "Hello ASP.NET Core". При этом в качестве значения может выступать любой объект. И после этому мы можем его использовать в представлении:

```csharp
@{
    ViewData["Title"] = "Index";
}
<h2>@ViewData["Title"].</h2>
<h3>@ViewData["Message"]</h3>

```

Причем не обязательно устанавливать все объекты во ViewData в контроллере. Так, в данном случае объект с ключом "Title" устанавливается непосредственно в представлении.

**ViewBag**

ViewBag во многом подобен ViewData. Он позволяет определить различные свойства и присвоить им любое значение. Так, мы могли бы переписать предыдущий пример следующим образом:

```csharp
public IActionResult Index()
{
    ViewBag.Message = "Hello ASP.NET Core";
 
    return View();
}
```

И таким же образом нам надо было бы изменить представление:

```csharp
@{
    ViewBag.Title = "Index";
}
<h2>@ViewBag.Title.</h2>
<h3>@ViewBag.Message</h3>
```
И не важно, что изначально объект ViewBag не содержит никакого свойства Message, оно определяется динамически. При этом свойства ViewBag могут содержать не только простые объекты типа string или int, но и сложные данные. Например, передадим список:

```csharp
public IActionResult Index()
{
    ViewBag.Countries = new List<string> { "Бразилия", "Аргентина", "Уругвай", "Чили" };
    return View();
}
```

И также в представлении мы можем получить этот список:

```csharp
<h3>В списке @ViewBag.Countries.Count элемента:</h3>
 
@foreach(string country in ViewBag.Countries)
{
    <p>@country</p>
}
```
    
Правда, чтобы выполнять различные операции, может потребоваться приведение типов, как в данном случае.

**Модель представления**

Модель представления является во многих случаях более предпочтительным способом для передачи данных в представление. Для передачи данных в представление используется одна из версий метода View:

```csharp
public IActionResult Index()
{
    List<string> countries = new List<string> { "Бразилия", "Аргентина", "Уругвай", "Чили" };
    return View(countries);
}
```

В метод View передается список, поэтому моделью представления Index.cshtml будет тип List<string> (либо IEnumerable<string>). И теперь в представлении мы можем написать так:

```csharp
@model List<string>
@{
    ViewBag.Title = "Index";
}
    
<h3>В списке @Model.Count элемента</h3>
@foreach(string country in Model)
{
    <p>@country</p>
}
```    
    
В самом начале представления с помощью директивы @model устанавливается модель представления. Тип модели должен совпадать с типом объекта, который передается в метод View() в контроллере. Установка модели указывает, что объект Model теперь будет представлять объект List<string> или список. И мы сможем использовать Model в качестве списка.

Представления, для которых определена модель, еще называют строго типизированными или strongly-typed views.

# Секции

Кроме метода ```RenderBody()```, который вставляет освновное содержимое представлений, мастер-страниц может также использовать специальный метод ```RenderSection()``` для вставки секций. Мастер-страница может иметь несколько секций, куда представления могут поместить свое содержимое. Например, добавим к мастер-странице ```_Master.cshtml``` секцию ```footer```:

```csharp
<!DOCTYPE html>
 
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>@ViewBag.Title</title>
</head>
<body>
    <div>
        @RenderBody()
    </div>
    <footer>@RenderSection("Footer")</footer>
</body>
</html>
```    
    
Теперь при запуске предыдущего представления ```Index``` мы получим ошибку, так как секция ```Footer``` не определена. По умолчанию представление должно передавать содержание для каждой секции мастер-страницы. Поэтому добавим вниз представления ```Index``` секцию ```footer```. Это мы можем сделать с помощью выражения ```@section```:

```csharp
@{
    ViewData["Title"] = "Home Page";
    Layout = "~/Views/_Master.cshtml";
}
<h2>Представление Index.cshtml</h2>
 
@section Footer {
    Все права защищены. Site Corp.
}
```    
    
Но при таком подходе, если у нас есть куча представлений, и мы вдруг захотели определить новую секцию на мастер-странице, нам придется изменить все имеющиеся представления, что не очень удобно. В этом случае мы можем воспользоваться одним из вариантов гибкой настройки секций.

```Первый``` вариант заключается в использовании перегруженной версии метода ```RenderSection```, которая позволяет указать, что данную секцию не обязательно определять в представлении. Чтобы отметить секцию ```Footer``` в качестве необязательной, надо передать в метод в качестве второго параметра значение false:1

<footer>@RenderSection("Footer", false)</footer>

    
```Второй``` вариант позволяет задать содержание секции по умолчанию, если данная секция не определена в представлении:

```html
<footer>
    @if (IsSectionDefined("Footer"))
    {
        @RenderSection("Footer")
    }
    else
    {
        <span>Содержание элемента footer по умолчанию.</span>
    }
</footer>
```
    
# Частичные представления
 
В приложениях на ASP.NET MVC кроме обычных представлений и мастер-страниц можно также использовать частичные представления или partial views. Их отличительной особенностью является то, что их можно встраивать в другие обычные представления. Частичные представления могут использоваться также как и обычные, однако наиболее удобной областью их использования является рендеринг результатов AJAX-запроса. По своему действию частичные представления похожи на секции, только их код выносится в отдельные файлы.

Частичные представления полезны для создания различных панелей веб-страницы, например, панели меню, блока входа на сайт, каких-то других блоков.

За рендеринг частичных представлений отвечает объект ```PartialViewResult```, который возвращается методом ```PartialView```. Итак, определим в контроллере ```HomeController``` новое действие GetMessage:

```csharp
public class HomeController : Controller
{
    public ActionResult GetMessage()
    {
        return PartialView("_GetMessage");
    }
    //....................
}
```
    
Теперь добавим в папку ```Views/Home``` новое представление ```_GetMessage.cshtml```, в котором будет простенькое содержимое:

```csharp
<h3>Частичное представление</h3>
```
Теперь обратимся к методу GetMessage как к обычному действию контроллера, и оно нам вернет частичное представление:

Частичные представления в ASP.NET MVC Core
По своему действию частичное представление похоже на обычное, только для него по умолчанию не определяется мастер-страница.

Встраивание частичного представления в обычное
Теперь рассмотрим, как мы можем встраивать частичные представления в обычные. Для этого изменим представление Index.cshtml:

```csharp
@{
    ViewData["Title"] = "Home Page";
}
<h2>Представление Index.cshtml</h2>
 
@await Html.PartialAsync("_GetMessage")
```
    
Метод ```Html.PartialAsync()``` встраивает код частичного представления в обычное. Он является асинхронным и возвращает объект ```IHtmlContent```, который представляет html-содержимое и который обернут в объект ```Task<TResult>```. В качестве параметра в метод передается имя представления:

Кроме метода ```Html.PartialAsync()``` частичное представление можно встроить с помощью другого метода - ```Html.RenderPartialAsync```. Этот метод также принимает имя представления, только он используется не в строчных выражениях кода ```Razor```, а в блоке кода, то есть обрамляется фигурными скобками:

```csharp
@{await Html.RenderPartialAsync("_GetMessage");}
```
```Html.RenderPartialAsync``` напрямую пишет вывод в выходной поток в асинхронном режиме, поэтому может работать чуть быстрее, чем ```Html.PartialAsync```.

Одна из перегруженных версий методов ```Html.PartialAsync / Html.RenderPartialAsync``` позволяет передать модель в частичное представление. В итоге у нас получится стандартное строго типизированное представление. Например, в качестве второго параметра список строк:

```@await Html.PartialAsync("_GetMessage", new List<string> { "Lumia 950", "iPhone 6S", "Samsung Galaxy s 6", "LG G 4" })```
Поскольку здесь в частичное представление передается список строк, то мы можем использовать модель ```List<string>```, чтобы получить этот список. Теперь изменим частичное представление ```_GetMessage.cshtml```:

```csharp
@model IEnumerable<string>
<h3>Список моделей</h3>
<ul>
    @foreach(string s in Model)
    {
        <li>@s</li>
    }
</ul>
```
 
Кроме обычных представлений и мастер-страниц в ASP.NET Core MVC также можно использовать частичные представления или partial views. Их отличительной особенностью является то, что их можно встраивать в другие обычные представления. Частичные представления могут использоваться также как и обычные, однако наиболее удобной областью их использования является рендеринг результатов AJAX-запроса. По своему действию частичные представления похожи на секции, которые использовались в прошлой теме, только их код выносится в отдельные файлы.

Частичные представления полезны для создания различных панелей веб-страницы, например, панели меню, блока входа на сайт, каких-то других блоков.

За рендеринг частичных представлений отвечает объект ```PartialViewResult```, который возвращается методом ```PartialView()```.

**Замечание**: по своему действию частичное представление похоже на обычное, только для него по умолчанию не определяется мастер-страница.


```csharp
@await Html.PartialAsync("_GetMessage")
```    
    
    
    
# Работа с формами и html
https://metanit.com/sharp/aspnetmvc/3.8.php

     
# Null условная проверка с объединением

```csharp
string name = р?.Name ?? "No name";
```

# Настройка Postman для localhost

Finally fixed it by turning off File > Settings > General > SSL Certificate Verification

# Компоненты представлений

```View Component``` или компонент представлений представляет код, который объединяет логику на языке C# и связанную с ней разметку ```Razor``` и который решает какую-то определенную задачу, например, создание динамических меню, облако тегов, панель входа на сайт, корзина покупок и так далее.

```View Component``` состоит из двух частей: класса на C# и частичного представления, которое вызывает методы этого класса. При этом ```View Component``` не может обрабатывать HTTP-запросы, а генерируемый им контент включается в код родительского представления, в котором вызывается компонент.

```View Component``` поддерживает внедрение зависимостей, поэтому в коде ```View component``` можно получить зависимости из провайдера сервисов. В то же время комопонент не является частью жизненного цикла контроллера, поэтому к нему нельзя применять фильтры.

# Создание компоненты 
    
Папка ```Components```
    
```Csharpp
using Microsoft.AspNetCore.Mvc;

namespace UserApp.Components
{
    public class Timer : ViewComponent
    {
        /*  public async Task<string> InvokeAsync()
        {
            return $"Текущее время: {DateTime.Now.ToString("hh:mm:ss")}";
        }*/

        public IViewComponentResult Invoke(UserApp.Models.User user )
        {
            return View("Index", user);
        }
    }
}   
```
 
# Вызов компонента в представлении
    
Теперь создадим представление, которое будет выводить переданные из компонента данные. Если при вызове метода ```View``` в компоненте имя представления явным образом не указано, то в качестве представления по умолчанию используется файл ```Default.cshtml```. Для поиска файла представления ```Razor``` будет просматривать следующие пути в порядке приоритета:

- Views/Название_Контроллера/Components/Название_Компонента/Название_Представления.cshtml

- Views/Shared/Components/Название_Компонента/Название_Представления.cshtml
    
```Csharp
@{

    User user = new UserApp.Models.User() { Id = 1, Name = "1", Age = 3 };
}

@await Component.InvokeAsync("Timer", new { user = user })

```

# Второй способ

```Csharp
<vc:timer user = "user" />

```
 Однако перед использованием тега нобходимо зарегистрировать наш компонент в представлении качестве tag-хелпера с помощью директивы @addTagHelper:   
 
 ```Csharp
@addTagHelper *, [Название_сборки_проекта]
 ```
 
При использовании тег-хелпера параметры компонента определяются как атрибуты тега, которым присваивается необходимое значение. Причем если у нас исползуется camelcase, при котором каждое подслово в составе составного слова пишется с большой буквы, например, includeSeconds, то в названии атрибута все подслова разделяются дефисом и начинаются со строчной буквы.
    
    







    
