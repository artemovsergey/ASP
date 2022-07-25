# ASP.Net Core

## Tехнологии

### .Net Core 6
###  EntityFrameworkCore (6.0.7)


### Быстрое создание прототипа MVC

dotnet new mvc --output TestMVC
cd TestMVC
mkdir Models
### Установить CLI-инструмент Scaffolding
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design

dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

### Создать модель

```Csharp
namespace TestMVC.Models
{
    public class Beer{
        public int Id { get; set; }
        public string Name { get; set; }
        public int Rating { get; set; }
    }
}
```

### Создание контекста BeerContext c EntityFrameworkCore

```Csharp
using Microsoft.EntityFrameworkCore;
namespace TestMVC.Data
{
    public class BeerContext : DbContext
    {
        public DbSet<TestMVC.Models.Beer> Beers { get; set; }
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            //Don't hardcode like this in a real app:
            optionsBuilder.UseSqlite("Data Source=beers.db");
        }
    }
}
```
### Добавить служюу
```Csharp
builder.Services.AddDbContext<BeerContext>();
```
### Сделать скаффолдинг
```Csharp
dotnet aspnet-codegenerator controller --controllerName Home --model Beer --dataContext BeerContext --useDefaultLayout -outDir Controllers -f --useSqlite
```
### Выполнить миграции в базу данных
```Csharp
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet run
```

**Замечание**: это можно только для тренировки


Алгоритм по книге

1. Создали пустой проект
2. Настройка структуры папок по mvc
3. Создание проекта тестов в решении через xUnit ( moq)
4. Создание модели предметной области
5. Контроллер получает объект через интерфейс. То есть логики в контроллере пока нет. Доступ к данным идёт через интерфейс
6. Подготовка представления: шаблон представления
7. Подготовка EF , инструменты командной строки EF, создание миграции, контекста, SQL Server
8. Подготовить начальные данные
9. Пагинация
10. Категории ( компоненты представлений)
11. Создание корзины , сессии
12. Оформление заказа
13. Администрирование, TempData
14. Защита Identity
15. Развертывание приложения на Azure
16. Работа с Visual Studio Code ( Linux)

Вывод: 
1. нужен английский, чтобы читать документацию и книги, 
2. нужен курс по фронтенду и созданию интерфейсов и дизайна
3. Нужен пакетный клиентский менеджер npm, webpack  для asp core


Задачи: 

- надо сделать простое CRUD приложение
- сделать интерфейс Bootstrap
- сделать css,javascript свои, может быть ajax
- второй проект для TDD
- git сразу
- deploy сразу
- база данных postgres
- сделать такое же api
- интегрировать api в приложение
- сделать мобильный клиент (можно на разных технологиях) и подключить через api
- сделать WPF + api
- база данных для api postgres
- система аутентификации
- привязка электронной почты
- механизм для генерации базы данных



---

### Вопрсы
1. Как вызвать у модели статический метод в контроллере?
2. Лучше использовать чистую html разметку или tag-хэлперы?
3. Есть ли скаффолд?
---
### Предложения

1. Начать с повторения html, css, bootstrap. Нужен курс по фронтенту
2. Статические переменные класса - это способ хранить состояние объекта в памяти (во время работы приложения)


### Теория

### ViewModel
Правильно использовать строгую типизацию представлений. Используя ViewBag или ViewData, наше представление теряет строгую типизацию.  Лучше всего использовать ViewModel.

Viewmodel – содержит поля, которые нужны для представлений. Это модель определённого View, или сразу для нескольких представлений. Они могут иметь логику для валидации, используя аннотацию данных для проверки моделей. ViewModel помогает реализовать строгую типизацию представления. Может быть нескольких сущностей в нескольких моделей.

### Проектирование модели данных

Модель - это представление реальных объектов, процессов и правил, которые определяют сферу приложения, известную как предметная
область. 

Модель, которую часто называют моделью предметной области, содержит объекты С# (или объекты предметной области), образующие "вселенную" приложения, и методы. позволяющие манипулировать ими. Представления и контроллеры открывают доступ клиентам к предметной области в согласованной манере, и любое корректно разработанное приложение MVC начинается с хорошо спроектированной модели, котора  затем служит центральным узлом при добавлении контроллеров и представлений. 

### Как работают дескрипторы при связывании данных из формы

Для каждого свойства класса модели GuestResponse определены элементы label и input (или элемент select в случае свойства WillAttend). Каждый элемент ассоциирован со свойством модели с применением еще одного атрибута вспомогательной функции дескриптора - asp-for.  Атрибуты вспомогательных функций дескрипторов конфигурируют элементы, чтобы привязать их к объекту модели.

### Запрос

Привязка модели освобождает нас от решения утомительной и подверженной ошибкам задачи по инспектированию НТТР-запроса и извлечению всех требующихся значений данных, но (что самое важное) при желании мы могли бы обрабатывать запрос вручную, поскольку MVC обеспечивает легкий доступ ко всем данным запроса. Ничто не скрыто от разработчика, но есть несколько удобных средств, которые упрощают работу с НТТР и HTML; тем не менее, использовать их вовсе не обязательно. 

### wwwroot

В проектах MVC соглашению статическое содержмое, доставляемое клиентам  в папку wwwroot, папки которой организованы по типу содержимого, так что таблицы стилей CSS находятся в папке wwwroot/css , файлы JavaScript - в папке wwwroot /js

Стандартная конфигурация ASP.NET включает поддержку обслуживания статического содержимого, такого как изображения, таблицы стилей CSS и файлы JavaScript, которая автоматически отображает запросы на папку wwwroot. 

По соглашению пакеты CSS и JavaScгipt от независимых поставщиков устанавливаются в папку wwwroot /lib

---

### Про дублирование кода

Инфраструктура MVC предлагает несколько средтв, помогающих сократить дублирование. 
К таким средтвам относятся:
- компоновки Razor
- частичные представления
- компоненты представлений. 


###  Про паттерн  MVC

Совет. Многих разработчиков, только приступивших к ознакомлению с паттерном MVC, приводит в замешательство идея помещения логики в модель данных из-за их уверенности
в том, что целью паттерна MVC является отделение данных от логики. Это заблуждение:
цель паперна MVC  -  разделение приложения на три функциональных области, каждая
из которых может содержать и логику, и данные. Цель не в том, чтобы устранить логику
из модели. Наоборот, цель в том, чтобы гарантировать наличие в модели только логики,
предназначенной для создания и управления данными модели.

### Службы
Службы ASP.NEТ - это объекты. которые предоставляют функциональность другим частям приложения.


## Program.cs для API

```Csharp
using WebAPI.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.InMemory;


var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle


/*
 
В ASP.NET Core службы (такие как контекст базы данных) должны быть зарегистрированы с помощью 
контейнера внедрения зависимостей. Контейнер предоставляет службу контроллерам.

 */

string connection = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<UserContext>(options => options.UseSqlServer(connection));

//builder.Services.AddDbContext<UserContext>(opt =>opt.UseInMemoryDatabase("UsersDatabase"));
// Добавляет контекст базы данных в контейнер внедрения зависимостей.
// Указывает, что контекст базы данных будет использовать базу данных в памяти.

builder.Services.AddEndpointsApiExplorer();

// тестирование api
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    
    // тестирвоание api 
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();

```

### API контроллер для модели User

```Csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using WebAPI.Models;

namespace WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UsersController : ControllerBase
    {
        private readonly UserContext _context;

        public UsersController(UserContext context)
        {
            _context = context;
        }



        // GET: api/Users
        [HttpGet]
        public async Task<ActionResult<IEnumerable<User>>> GetUsers()
        {
            return await _context.Users.ToListAsync();
        }



        // GET: api/Users/5
        [HttpGet("{id}")]
        public async Task<ActionResult<User>> GetUser(int id)
        {
            var user = await _context.Users.FindAsync(id);

            if (user == null)
            {
                return NotFound();
            }

            return user;
        }


        // PUT: api/TodoItems/5
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateUsers(int id, User user)
        {
            if (id != user.Id)
            {
                return BadRequest();
            }

            var current_user = await _context.Users.FindAsync(id);
            if (current_user == null)
            {
                return NotFound();
            }

            current_user.Name = user.Name;
            current_user.Age = user.Age;

            try
            {
                await _context.SaveChangesAsync();
            }
            catch
            {
                return NotFound();
            }

            return NoContent();
        }


        // DELETE: api/TodoItems/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteTodoItem(int id)
        {
            var todoItem = await _context.Users.FindAsync(id);
            if (todoItem == null)
            {
                return NotFound();
            }

            _context.Users.Remove(todoItem);
            await _context.SaveChangesAsync();

            return NoContent();
        }


        // POST: api/TodoItems
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPost]
        public async Task<ActionResult<User>> CreateTodoItem(User user)
        {
            var user1 = new User
            {
                Age = user.Age,
                Name = user.Name
            };

            _context.Users.Add(user1);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user1);



        }
    }
}
```

## Program.cs

```csharp

using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.SqlServer;
using MyProject.Models;

var builder = WebApplication.CreateBuilder(args);


builder.Services.AddMvc();
//builder.Services.AddControllersWithViews();
//builder.Services.AddMvc(options => options.EnableEndpointRouting = false);

// Подключение базы данных SQL Server
string connection = builder.Configuration.GetConnectionString("DefaultConnection");
//builder.Services.AddDbContext<TestStoreContext>(options => options.UseSqlServer(connection));


// Это значит совместное использоване объекта класса AppTimeService во всем приложении.
// Общая функциональность для приложения
builder.Services.AddSingleton<AppTimeService>();


//Какие преимущества? Нам не придется возиться с созданием объекта, удалением его или управлением этим объектом в коде наших страниц.


// Создание служю необходимых для управления сеансом
builder.Services.AddMemoryCache();
builder.Services.AddSession();

// Сервисы, это любой функционал, который мы хотим зарегистрировать, чтобы другие части приложения, могли его использовать (эл почта, бд…).

// Configure the HTTP request pipeline.

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

app.Run();
//

public class AppTimeService { };

```






## Модель с валидацией данных полей

Создать класс модели в папке Models

```csharp
using System.ComponentModel.DataAnnotations;

    public class User 
    {
        [Required(ErrorMessage = "Пожайлуста введите id пользователя")]
        public int Id { get; set; }
        
        [Required(ErrorMessage = "Пожайлуста введите имя пользователя")]
        public string Name { get; set; }

        [Required(ErrorMessage = "Пожайлуста введите возраст пользователя")]
        public int? Age { get; set; }
    }
```







### Проверка в контроллере

```Csharp
    if (ModelState.IsValid)
    {
        _allUsers.Add(user);
        return View("Users",_allUsers);
    } 
```



### Entity Framework Core. Создание контекста в Models

```csharp

using Microsoft.EntityFrameworkCore;
 
namespace MobileStore.Models
{
    public class MobileContext : DbContext
    {
        public DbSet<Phone> Users { get; set; }
       
        public MobileContext(DbContextOptions<MobileContext> options)
            : base(options)
        {
            Database.EnsureCreated();
        }
    }
}
```

### appsettings.json

По умолчанию у нас база данных отсутствуют. Поэтому в конструктор MobileContext определен вызов Database.EnsureCreated(), который при отсутствии базы данных автоматически создает ее. Если база данных уже есть, то ничего не происходит.
Чтобы подключаться к базе данных, нам надо задать параметры подключения. Для этого изменим файл appsettings.json. По умолчанию он содержит только настройки логгирования:

```JSON

{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=mobilestore;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}

```



### Файл _ViewStart

```Csharp
@{
    Layout = "_Layout";
}
```





### Файл _ViewImports

```Csharp
@using AppTest.Models;
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```



---


### Метод GET для вывода всех пользователей (Index)
```csharp

// Запрос на вывод все пользователей на /Home/index

[HttpGet]
public IActionResult Index()
{
    return View(db.Users.ToList());
}

```

### Метод GЕT на вывод одного пользователя (Show/id)

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

### Метод POST на добавление пользователя (create)

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

### Представление на добавление пользователя

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


## Стилизация Bootstrap

### Стили css для валидаций

```css

    .field-validation-error {color: #f00;}
    .field-validation-valid {display: none}
    .input-validation-error {border: 1px solid #f00;background-color: #fee;}
    .validation-summary-errors {font-weight: bold; color: #f00;}
    .validation-summary-valid {display: none;}

```

### Представление на редактирование пользователя

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









### Метод GET на редактирование пользователя (edit/id)

```csharp

[HttpGet]
// Home/AddUser
public IActionResult EditUser(int id) // просто отображаем форму
{
    User user = db.Users.Where(u => u.Id == id).FirstOrDefault();


    return View(user);
}

```

### Метод POST на редактирование пользователя (edit)

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

### Метод DELETE на удаление пользователя

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


### Index.cshtml

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

### Пример макета или мастер-страницы _Layout.cshtml

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

## Контекст базы данных

Для создания контекста в ```Program.cs```

```csharp
// Подключение базы данных SQL Server
string connection = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<AppContext>(options => options.UseSqlServer(connection));
```





### Получение данных из контекста запроса

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


## Получение данных отправленных форм

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



### Переадресация

Теория

Протокол HTTP поддерживает два типа переадресации:

постоянная переадресация. При постоянной переадресации сервер будет отправлять браузеру статусный код 301. При данном типе переадресации предполагается, что запрашиваемый документ окончательно перемещен в другое место. И после получения статусного кода 301 браузер может автоматически настраивать запросы на новый ресурс, даже если старый ресурс со временем перестанет применять переадресацию. Поэтому данный способ можно использовать, если вы полностью уверены, что документ на старое место уже не возвратится..

временная переадресация. При временной переадресации сервер будет отправлять браузеру статусный код 302. При этом считается, что запрашиваемый документ временно перемещен на другую страницу.

В обоих случаях для создания переадресации может использоваться объект RedirectResult, однако метод, возвращающий данный объект, будет отличаться.


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






### Отправка статусных кодов

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






### Отправка файлов
https://metanit.com/sharp/aspnet5/5.7.php

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











### Переопределение контроллеров

Как правило, для создания контроллера достаточно унаследовать свой класс от базового класса Controller. Однако если нам необходимо, чтобы наши контроллеры реализовали некоторую общую логику, мы можем определить свой базовый класс контроллера и уже от него наследовать остальные контроллеры. Либо мы также можем переопределить некоторые методы базового класса Controller, если они нас не устраивают.

Что мы можем в контроллере переопределить? В базовом классе Controller среди всех прочих методов есть три интересных метода:

Метод OnActionExecuting() выполняется при вызове метода контроллера до его непосредственного выполнения.

Метод OnActionExecuted() выполняется после выполнения метода контроллера.

Метод OnActionExecutionAsync() представляет асинхронную версию метода OnActionExecuting().

**Замечание**: это похоже на callback или фильтры перед действиями контроллеров в Rails


### Передача зависимостей в контроллер

Как и любой класс, контроллер может получать сервисы приложения через механизм dependency injection. В контроллере это можно делать следующими способами:

- Через конструктор
- Через параметр метода, к которому применяется атрибут FromServices
- Через свойство HttpContext.RequestServices

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

```

В данном случае определен интерфейс ITimeService и его реализация SimpleTimeService. И в приложении происходит регистрация сервиса ITimeService.

Передача через конструктор

Когда приходит запрос к контроллеру, инфраструктура MVC вызывает провайдер сервисов для создания объекта HomeController. Провайдер сервисов проверят конструктор класса HomeController на наличие зависимостей. Затем создает объекты для всех используемых зависимостей и передает их в конструкторо HomeController для создания объекта контроллера, который затем обрабатывает запрос.

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

Приложение получает запрос к методу контроллера HomeController
Фреймворк MVC обращается к провайдеру сервисов для создания объекта контроллера HomeController
Провайдер сервисов смотрит на конструктор класса HomeController и видит, что там имеется зависимость от интерфейса ITimeService
Провайдер сервисов среди зарегистрированных зависимостей ищет класс, который представляет реализацию интерфейса ITimeService
Если нужная зависимость найдена, то провайдер сервисов создает объект класса, который реализует интерфейс ITimeService
Затем провайдер сервисов создает объект HomeController, передавая в его конструктор ранее созданную реализацию ITimeService
В конце провайдер сервисов возвращает созданный объект HomeController инфраструктуре MVC, которая использует контроллер для обработки запроса

Передача зависимостей в методы. FromServices

Иногда зависимость используется только в одном методе. И в этом случае нет необходимости передавать ее в контроллер, поскольку она напрямую может быть внедрена в сам метод, который ее использует. Для передачи зависимости в метод применяется атрибут [FromServices]:

```Csharp
public string Index([FromServices] ITimeService timeService)
{
    return timeService.Time;
}
```
Атрибут FromServices предоставляется инфраструктурой MVC - он определен в пространстве имен Microsoft.AspNetCore.Mvc.

HttpContext.RequestServices

В методах контроллера можно обращаться к объекту контекста запроса через свойство HttpContext, а через свойство HttpContext.RequestServices можно получить все зарегистрированные в приложении сервисы:

```Csharp
public string Index()
{
    ITimeService? timeService = HttpContext.RequestServices.GetService<ITimeService>();
    return timeService?.Time ?? "Undefined";
}
```




### Контекст контроллера
https://metanit.com/sharp/aspnet5/5.9.php





### Передача зависимотей в контроллер
https://metanit.com/sharp/aspnet5/5.10.php






















## Представление Views

### Представления Razor

**Замечание**: полноценной html-страницей представление все равно не является, потому что во время выполнения эти представления компилируются в сборки и уже затем используются для генерации html-страниц, которые видит пользователь в своем браузере.

Для хранения представлений в проекте ASP.NET MVC предназначена папка ```Views```:

Представления в ASP.NET MVC Core
В этой папке уже есть некоторая подструктура. Во-первых, как правило, для каждого контроллера в проекте создается подкаталог в папке Views, который называется по имени контроллера и который хранит представления, используемые методами данного контроллера. Так, по умолчанию имеется контроллер HomeController и для него в папке Views есть подкаталог Home с представлениями для методов контроллера HomeController.

Также здесь есть папка Shared, которая хранит общие представления для всех контроллеров. По умолчанию это файлы _Layout.cshtml (используется в качестве мастер-страницы), Error.cshtml (использутся для отображения ошибок) и _ValidationScripsPartial.cshtml (частичное представление, которое подключает скрипты валидации формы).

И в корне каталога Views также можно найти два файла _ViewImports.cshtml и _ViewStart.cshtml. Эти файлы содержат код, который автоматически добавляется ко всем представлениям. _ViewImports.cshtml устанавливает некоторые общие для всех представлений пространства имен, а _ViewStart.cshtml устанавливает общую мастер-страницу.


### Применение мастер-страницы

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






### Частичное представление
```csharp
@await Html.PartialAsync("_GetMessage")
```

### Передача данных в представление

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
 
<p>Random text.</p>
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
 
 
<p>Random text.</p>
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

Модель представления в ASP.NET MVC Core
Представления, для которых определена модель, еще называют строго типизированными или strongly-typed views.


### Секции

Кроме метода RenderBody(), который вставляет освновное содержимое представлений, мастер-страниц может также использовать специальный метод RenderSection() для вставки секций. Мастер-страница может иметь несколько секций, куда представления могут поместить свое содержимое. Например, добавим к мастер-странице _Master.cshtml секцию footer:

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
    
Теперь при запуске предыдущего представления Index мы получим ошибку, так как секция Footer не определена. По умолчанию представление должно передавать содержание для каждой секции мастер-страницы. Поэтому добавим вниз представления Index секцию footer. Это мы можем сделать с помощью выражения @section:

    
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

Первый вариант заключается в использовании перегруженной версии метода RenderSection, которая позволяет указать, что данную секцию не обязательно определять в представлении. Чтобы отметить секцию Footer в качестве необязательной, надо передать в метод в качестве второго параметра значение false:

1
<footer>@RenderSection("Footer", false)</footer>
Второй вариант позволяет задать содержание секции по умолчанию, если данная секция не определена в представлении:

```csharp
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
    
### Частичные представления
 
В приложениях на ASP.NET MVC кроме обычных представлений и мастер-страниц можно также использовать частичные представления или partial views. Их отличительной особенностью является то, что их можно встраивать в другие обычные представления. Частичные представления могут использоваться также как и обычные, однако наиболее удобной областью их использования является рендеринг результатов AJAX-запроса. По своему действию частичные представления похожи на секции, которые использовались в прошлой теме, только их код выносится в отдельные файлы.

Частичные представления полезны для создания различных панелей веб-страницы, например, панели меню, блока входа на сайт, каких-то других блоков.

За рендеринг частичных представлений отвечает объект PartialViewResult, который возвращается методом PartialView. Итак, определим в контроллере HomeController новое действие GetMessage:

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
    
Теперь добавим в папку Views/Home новое представление _GetMessage.cshtml, в котором будет простенькое содержимое:

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
    
    Метод Html.PartialAsync() встраивает код частичного представления в обычное. Он является асинхронным и возвращает объект IHtmlContent, который представляет html-содержимое и который обернут в объект Task<TResult>. В качестве параметра в метод передается имя представления:

Partial Views in ASP.NET MVC Core
Обращения к методу GetMessage() в контроллере при этом не происходит.

Кроме метода Html.PartialAsync() частичное представление можно встроить с помощью другого метода - Html.RenderPartialAsync. Этот метод также принимает имя представления, только он используется не в строчных выражениях кода Razor, а в блоке кода, то есть обрамляется фигурными скобками:

```csharp
@{await Html.RenderPartialAsync("_GetMessage");}
```
    Html.RenderPartialAsync напрямую пишет вывод в выходной поток в асинхронном режиме, поэтому может работать чуть быстрее, чем Html.PartialAsync.

Одна из перегруженных версий методов Html.PartialAsync / Html.RenderPartialAsync позволяет передать модель в частичное представление. В итоге у нас получится стандартное строго типизированное представление. Например, в качестве второго параметра список строк:

1
@await Html.PartialAsync("_GetMessage", new List<string> { "Lumia 950", "iPhone 6S", "Samsung Galaxy s 6", "LG G 4" })
Поскольку здесь в частичное представление передается список строк, то мы можем использовать модель List<string>, чтобы получить этот список. Теперь изменим частичное представление _GetMessage.cshtml:

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

 
 
 
 
 
 
 
### Маршрутизация на основе аттрибутов

Фреймворк MVC позволяет использовать в приложении маршрутизацию на основе атрибутов. Такой тип маршрутизации еще называется Attribute-Based Routing. Атрибуты предоставляют более гибкий способ определения маршрутов. Маршруты, определенные с помощью атрибутов, имеют приоритет по сравнению с маршрутами, определенными в классе Startup.

Маршрутизация на основе атрибутов может применяться независимо от того, какой компонент отвечает в приложении за маршрутизацию - RouterMiddleware или EndpointRoitingMiddleware+EndpointMiddleware.

Для определения маршрута, необходимо использовать атрибут [Route]:

```csharp
public class HomeController : Controller
{
    [Route("Home/Index")]
    public IActionResult Index()
    {
        return Content("ASP.NET Core на metanit.com");
    }
    [Route("About")]
    public IActionResult About()
    {
        return Content("About site");
    }
}
```
    
В данном случае метод Index будет обрабатывать запросы по адресу "Home/Index", а метод About по адресу "About".

Attribute-Based Routing in ASP.NET Core
Если в проекте планируется использовать только маршрутизацию на основе атрибутов, то в классе Startup мы можем не определять никаких маршрутов. Например, при использовании EndpointMiddleware и конечных точек можно применить метод MapControllers:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    app.UseStaticFiles();
 
    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();     // нет определенных маршрутов
    });
}
Аналогично можно не определять маршруты и при использовании RouterMiddleware:
```
    
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
 
    app.UseStaticFiles();
    app.UseMvc();           // нет определенных маршрутов
}
Но также мы можем комбинировать способы маршрутизации. При чем здесь надо учитывать, что маршрутизация на основе атрибутов имеет больший приоритет. Например, определим следующее действие контроллера:
```
    
```csharp
public class HomeController : Controller
{
    [Route("homepage")]
    public IActionResult Index()
    {
        return Content("Hello ASP.NET MVC на https://metanit.com");
    }
}
```
В качестве параметра атрибут Route принимает шаблон URL, с которым будет сопоставляться запрошенный адрес. И даже если у нас определен стандартный маршрут в классе Startup:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    app.UseStaticFiles();
 
    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        // определение маршрутов
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```
    
Запрос типа http://localhost:xxxx/home/index работать не будет, так как мы явно указали с помощью атрибута, что метод Index контроллера Home будет обрабатывать только запросы http://localhost:xxxx/homepage

 
 
 
 
 
 
### Атрибуты маршрутизации в ASP.NET MVC Core
 
 
Определение маршрутов с помощью атрибутов подчиняется тем же правилам, что определение маршрутов в классе Startup. Например, используем параметры и ограничения:

```csharp
public class HomeController : Controller
{
    [Route("{id:int}/{name:maxlength(10)}")]
    public IActionResult Test(int id, string name)
    {
        return Content($" id={id} | name={name}");
    }
}
```
И к такому методу мы сможем обратиться с запросом типа http://localhost:xxxx/10/lumia.

Атрибуты контроллеров и маршрутизация в ASP.NET MVC Core
Или другой пример - применение необязательного параметра:

```csharp
public class HomeController : Controller
{
    [Route("Home/Index/{id?}")]
    public IActionResult Test(int? id)
    {
        if(id!=null)
            return Content($"Параметр id={id}");
        else
            return Content($"Параметр id неопределен");
    }
}
```
    
И к такому методу мы сможем обратиться с запросом типа http://localhost:xxxx/home/index или добавить параметр http://localhost:xxxx/home/index/5.

 
 
### Использование префиксов

 Допустим, у нас есть несколько методов, для которых определен свой маршрут, и мы хотим, чтобы эти маршруты начинались с одного определенного префикса. Например:

```csharp
public class HomeController : Controller
{
    [Route("main/index/{name}")]
    public IActionResult Index(string name)
    {
        return Content(name);
    }
    [Route("main/{id:int}/{name:maxlength(10)}")]
    public IActionResult Test(int id, string name)
    {
        return Content($" id={id} | name={name}");
    }
}
```
    
Вместо того, чтобы добавлять префикс "main" к каждому маршруту, мы могли бы определить его глобально для всего контроллера:

    
```csharp
[Route("main")]
public class HomeController : Controller
{
    [Route("index/{name}")]
    public IActionResult Index(string name)
    {
        return Content(name);
    }
    [Route("{id:int}/{name:maxlength(10)}")]
    public IActionResult Test(int id, string name)
    {
        return Content($" id={id} | name={name}");
    }
}
```
    
 
### Параметры controller и action
 
От всех параметров шаблона маршрутов в атрибутах отличаются два параметра controller и action, которые ссылаются соответственно на контроллер и его действие. При использовании их надо помещать в квадратные скобки, а не в фигурные, как другие параметры:

```csharp
public class HomeController : Controller
{
    [Route("[controller]/[action]")]
    public IActionResult Index()
    {
        var controller = RouteData.Values["controller"].ToString();
        var action = RouteData.Values["action"].ToString();
        return Content($"controller: {controller} | action: {action}");
    }
}
```
    
Допустимо также использование их по отдельности или добавление к ним других параметров в фигурных скобках:

```csharp
[Route("[controller]")]
[Route("[action]")]
[Route("[controller]/[action]/{id?}")]
```
    Множественные маршруты
С помощью атрибутов можно задать несколько маршрутов для одного метода. Например:

```csharp
[Route("[controller]")]
public class HomeController : Controller
{
   [Route("")]     // сопоставляется с Home
   [Route("Index")] // сопоставляется с Home/Index
   public IActionResult Index()
   {
        return View();
   }
}
```
    
Также для контроллера можно задать сразу несколько маршрутов:
    
```csharp
[Route("Store")]
[Route("[controller]")]
public class HomeController : Controller
{
   [Route("Main")]     // сопоставляется с Home/Main, либо с Store/Main
   [Route("Index")] // сопоставляется с Home/Index, либо с Store/Index
   public IActionResult Index()
   {
        return View();
   }
}
```
    
    
   
### Применение дескриптеров  в форме

 ```csharp
          <form asp-action="rsvpform" methdod="post">
                
                <p>
                    <label asp-for="Name">Your Name</label>
                    <input asp-for="Name"/>
                </p>

                <p>
                    <label asp-for="Email">Ваша почта</label>
                    <input asp-for="Email"/>
                </p>

                <p>
                    <label asp-for="Phone">Телефон</label>
                    <input asp-for="Phone"/>
                </p>
                <p>
                    <label> WillAttend</label>
                    <select asp-for="WillAttend">
                        <option value="Choose an option"></option>
                        <option value="true">Да</option>
                        <option value="false">Нет</option>
                    </select>
                </p>
                <button type="submit">Отправить</button>

            </form>
```

Каждый элемент ассоциирован со свойством модели с помощью дескриптора asp-for


---

 Цель паттерна MVC - разделение приложения на три функциональных области, каждая из которых может содержать и логиу и данные. Цель не в том, чтобы устранить логику из модели. Наоборот, цель в том, чтобы гарантировать наличие в модели только логики предназначенной для создания и управления данными

 ---

 
### Null условная проверка с объединением
```csharp

string name = р?.Name ?? "No name"; // null условная операция с объединением

```

 
 
## Сопоставление с образцом

```csharp

if (data is decimal d) { }

```
**Замечание**: сопоставлнеие с образцом также работает с оператором ```switch```

---

 
 
### Применение лямбда выражений

```csharp
public ViewResult Index() {
return View()
}
 
или
 
```csharp
pubclic ViewResult Index() => View()
}

```

 
 
 
### Подключение Bootstrap и js

В шаблоне сделать ссылку
```html <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.css"/> <script src = "js/"></script>```

 
 
 
 
 
 
### Настройка Postman для localhost

Finally fixed it by turning off File > Settings > General > SSL Certificate Verification

 
 
 
 

### Создаем миграции с помощью консольных команд

```csharp
 
dotnet ef migrations add Itinial
dotnet ef database update

```

 
 
 
### Области
 
    https://metanit.com/sharp/aspnet5/11.9.php

### Модели
    https://metanit.com/sharp/aspnet5/8.1.php

    
### Html-хэлперы
    https://metanit.com/sharp/aspnet5/9.1.php

### Tag-хэлперы
    https://metanit.com/sharp/aspnet5/10.1.php

### Компоненты
    https://metanit.com/sharp/aspnet5/7.6.php

### Валидации моделей
    https://metanit.com/sharp/aspnet5/19.1.php

### Работа с Entity Framework. Сортировка, фильтрация, пагинация
    https://metanit.com/sharp/aspnet5/12.1.php

### Razor Pages - то же самое, что и MVC, только почти без контроллеров в чистом виде
    https://metanit.com/sharp/aspnet5/29.1.php

### Web API
    https://metanit.com/sharp/aspnet5/23.1.php

### Фильтры как в Rails
    https://metanit.com/sharp/aspnet5/18.1.php

### Применение аутентификации и авторизации средаствами ASP .NET Core
    https://metanit.com/sharp/aspnet5/15.1.php
    https://metanit.com/sharp/aspnet5/16.1.php

### Клиентская разработка
    https://metanit.com/sharp/aspnet5/13.6.php

### Кеширование
    https://metanit.com/sharp/aspnet5/14.1.php

### Сервер и публикация приложения
    https://metanit.com/sharp/aspnet5/2.7.php
    
### URL Rewriting
    https://metanit.com/sharp/aspnet5/24.1.php

### Глобализация и локализация
    https://metanit.com/sharp/aspnet5/28.1.php

### SignalR Core
    https://metanit.com/sharp/aspnet5/30.1.php
 
### Кроссдоменные запросы. CORS и кросс-доменные запросы
    https://metanit.com/sharp/aspnet5/31.1.php

### Тестирование
    https://metanit.com/sharp/aspnet5/22.1.php

### Dapper - альтернатива EF
    https://metanit.com/sharp/aspnet5/26.1.php

### Mongo DB
    https://metanit.com/sharp/aspnet5/27.1.php
    
### React.JS
    https://metanit.com/sharp/aspnet5/25.1.php

### Сервисы gRPC
    https://metanit.com/sharp/aspnet5/32.1.php
    
### Отправка email в ASP.NET Core
    https://metanit.com/sharp/aspnet5/21.1.php

### Загрузка файлов на сервер
    https://metanit.com/sharp/aspnet5/21.3.php
    

 
 
 
 ---
## Работа с Visual Code
## Создание приложения

Веб-приложение
```csharp
dotnet new webapp -o aspnetcoreapp
```
MVC

```csharp
dotnet new mvc -o aspnetcoreapp
```

Все шаблоны
```dotnet new --list```



## Установка доверия к сертификату разработки
```csharp
dotnet dev-certs https --trust
```

## Запуск приложения
```csharp
dotnet watch run

```
Можно запустить ```dotnet run```.

## Пример добавления пакетов Nugent

```csharp
dotnet new webapi -o TodoApi
cd TodoApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory
code -r ../TodoApi
```


    
    
