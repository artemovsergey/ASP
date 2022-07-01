# ASP Core 6

## Модель

Создать класс модели в папке Models

```asp
    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }

    }
```

## Контроллер

Создать класс контроллера в папке Controllers.

```СSharp

using Microsoft.AspNetCore.Mvc;

namespace Example.Controllers;

    public class HomeController : Controller
    {

        public IActionResult Index()
        {
            return View();
        }

    }

```
**Замечание**: можно создать отдельный файл класса с глобальными подключениями ```global using Microsoft.AspNetCore.Mvc ```

## Представление View

Создадим представление пустое Razor. Это файл с расширением cshtml в папке View. В представлении подключаем модель.
 Далее можно использовать переданный объект модели из action контроллера в представлении через @Model

- Example - название проекта
- Models - папка Models в проекте
- MyModel - название модели 

```asp
@model Example.Models.MyModel;
<h1>@Model.Message</h1>
```
## Program.cs

```asp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
builder.Services.AddMvc();
builder.Services.AddMvc(options => options.EnableEndpointRouting = false);


var app = builder.Build();
app.UseDeveloperExceptionPage(); // Ошибки при отладке
app.UseHttpsRedirection();
app.UseStaticFiles(); // доставка статического содержимого
app.UseStatusCodePages();
app.UseMvc();
app.UseMvcWithDefaultRoute();
app.UseRouting();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
app.MapDefaultControllerRoute();
app.Run();

```

## Entityt Framework Core

После определения моделей надо выбрать хранилище данных для этих моделей. Мы будем использовать MS SQL Server. Для работы с MS SQL Server компания Microsoft рекомендует использовать ORM-технологию Entity Framework, хотя ее использование необязательно. Мы также можем применять другие ORM-технологии или доступные средства ADO.NET. Преимущество фреймворка Entity Framework состоит в том, что он позволяет абстрагироваться от структуры конкретной базы данных и вести все операции с данными через модель.

Чтобы взаимодействовать с базой данных нам нужен контекст данных. Причем Entity Framework Core использует подход Code First, при котором нам надо сначала определить модели и контекст данных, а потом уже исходя и этих моделей и класса контекста будет создаваться бд и все ее таблицы.

```asp

using Microsoft.EntityFrameworkCore;
 
namespace MobileStore.Models
{
    public class MobileContext : DbContext
    {
        public DbSet<Phone> Phones { get; set; }
        public DbSet<Order> Orders { get; set; }
 
        public MobileContext(DbContextOptions<MobileContext> options)
            : base(options)
        {
            Database.EnsureCreated();
        }
    }
}

```

## Настройки подключения appsettings.json

По умолчанию у нас база данных отсутствуют. Поэтому в конструктор MobileContext определен вызов Database.EnsureCreated(), который при отсутствии базы данных автоматически создает ее. Если база данных уже есть, то ничего не происходит.
Чтобы подключаться к базе данных, нам надо задать параметры подключения. Для этого изменим файл appsettings.json. По умолчанию он содержит только настройки логгирования:

```json

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

## Настройка подключения в файле Program.cs

```asp
// Подключение базы данных SQL Server
string connection = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<MobileContext>(options => options.UseSqlServer(connection));
```

## Работа с контекстом

После того, как мы настроили подключение к базе данных мы можем работать через модели посредством контекста с базой данных
Например, в контроллере может быть следующий код

```asp

public class HomeController : Controller
{


    // Поступает запрос
    // Контроллер создает в конструкторе контекст базы данных

    MobileContext db;
    public HomeController(MobileContext context)
    {
        db = context;
    }


    public IActionResult Index()
    {
        return View(db.Phones.ToList());
    }
}

```

## Передача данных из контроллера в View

Представление получает доступ к данным, которые были переданы из метода контроллера

```asp
@model IEnumerable<Example.Models.Phone>
```
Первая строка устанавливает модель представления - та сущность, которая будет доступна в представлении через объект Model. В данном случае это объект IEnumerable<MobileStore.Models.Phone>, так как в контроллере в методе Index мы передаем список смартфонов, а список - объект List представляет интерфейс IEnumerable.

Теперь данные доступны посредством объекта @Model

**Замечание**: Каждый метод контроллера использует по умолчанию одноименный метод представления


## Мастер страница

```asp
@{
    Layout = null
}

Далее идет блок кода, в котором выражение Layout = null указывает, что мастер-страница (макет) не будет применяться к этому представлению

```


## Применнеие мастер-страницы

Применение мастер-страницы (макет). Файл _Layout.cshtml должен находиться в папке Views/Shared
```asp
@{
    Layout ="_Layout"; 
}
```

Пример макета Layout
```asp
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

## Частичное представление
```asp
@await Html.PartialAsync("_GetMessage")
```


## Класс контроллеров
При использовании контроллеров существуют некоторые условности. Во-первых, в проекте контроллеры помещаются в каталог Controllers. И во-вторых, по соглашениям об именовании названия контроллеров обычно оканчиваются на суффикс "Controller", остальная же часть до этого суффикса считается именем контроллера, например, HomeController. Но в принципе эти условности необязательны.

Но есть также и обязательные условности, которые предъявляются к контроллерам. В частности, класс контроллера должен удовлетворять как минимум одному из следующих условий:

Класс контроллера имеет суффикс "Controller"

public class HomeController
{  
    //............  
}
Класс контроллера наследуется от класса, который имеет суффикс "Controller"

public class Home : Controller
{
    //.............
}
К классу контроллера применяется атрибут [Controller]

[Controller]
public class Home
{
    //..................
}


Можно применять аттурибуты NonController, ActionName и NonAction


## Передача данных в контроллер
Вместе с запросом приложению могут приходить различные данные. И чтобы получить эти данные, мы можем использовать разные способы. Самым распространенным способом считается применение параметров.

Определение в методах контроллера параметров ничем не отличается от определения параметров в языке C#. Параметры могут представлять примитивные типы, как int или string, а могут представлять и более сложные классы.

Передавать значения для параметров можно различными способами. При отправке GET-запроса значения передаются через строку запроса. Стандартный get-запрос принимает примерно следующую форму: название_ресурса?параметр1=значение1&параметр2=значение2. Например, определим следующий метод:


Система привязки MVC, которую мы позже рассмотрим, сопоставляет параметры запроса и параметры метода по имени. То есть, если в строке запроса идет параметр a, то его значение будет передаваться именно параметру метода, который также называется a. При этом должно быть также соответствие по типу, то есть если параметр метода принимает числовое значение, то и через строку запроса надо передавать для этого параметра число, а не строку.

Post запросы
Чтобы система могла связать параметры метода и данные формы, необходимо, чтобы атрибуты name у полей формы соответствовали названиям параметров.



## Получение данных из контекста запроса

Параметры представляют самый простой способ получения данных, но в действительности нам необязательно их использовать. В контроллере доступен объект Request, у которого можно получить как данные строки запроса, так и данные отправленных форм.

Данные строки запроса доступны через свойство Request.Query, которое представляет объект IQueryCollection. Например:
public string Area()
{
    string altitudeString = Request.Query.FirstOrDefault(p => p.Key == "altitude").Value;
    int altitude = Int32.Parse(altitudeString);
 
    string heightString = Request.Query.FirstOrDefault(p => p.Key == "height").Value;
    int height = Int32.Parse(heightString);
 
    double square = altitude * height / 2;
    return $"Площадь треугольника с основанием {altitude} и высотой {height} равна {square}";
}
В данном случае метод Area обрабатывает GET-запросы, и мы можем к нему обратиться через запрос типа http://localhost:57086/Home/Area?altitude=20&height=4.


## Получение даннхы отправленных форм

Для получения данных отправленных форм можно использовать свойство Request.Form. Это свойство представляет объект IFormsCollection, но работает аналогично Request.Query:
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


## Результаты действий action

Однако в большинстве случаев нам не придется создавать свои классы результатов, потому что фреймворк ASP.NET MVC Core итак предоставляет довольно большое количество классов результатов для самых различных ситуаций:

ContentResult: пишет указанный контент напрямую в ответ в виде строки

EmptyResult: отправляет пустой ответ в виде статусного кода 200

public IActionResult GetVoid()
{
    return new EmptyResult();
}
Аналогичен следующему методу:

public void GetVoid()
{
}
NoContentResult: во многом похож на EmptyResult, также отправляет пустой ответ, только в виде статусного кода 204

public IActionResult GetVoid()
{
    return new NoContentResult();
}
FileResult: является базовым классом для всех объектов, которые пишут набор байтов в выходной поток. Предназначен для отправки файлов

FileContentResult: класс, производный от FileResult, пишет в ответ массив байтов

VirtualFileResult: также производный от FileResult класс, пишет в ответ файл, находящийся по заданному пути

PhysicalFileResult: также производный от FileResult класс, пишет в ответ файл, находящийся по заданному пути. Только в отличие от предыдущего класса использует физический путь, а не виртуальный.

FileStreamResult: класс, производный от FileResult, пишет бинарный поток в выходной ответ

ObjectResult: возвращает произвольный объект, как правило, применяется в качестве базового класса для других классов результатов. Но можно применять и самостоятельно:

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return new ObjectResult(new Person { Name="Tom", Age=35});
    }
}
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
StatusCodeResult: результат действия, который возвращает клиенту определенный статусный код HTTP

UnauthorizedResult: класс, производный от StatusCodeResult. Возвращает клиенту ответ в виде статусного кода HTTP 401, указывая, что пользователь не прошел авторизацию и не имеет прав доступа к запрошенному ресурсу.

NotFoundResult: производный от StatusCodeResult. Возвращает клиенту ответ в виде статусного кода HTTP 404, указывая, что запрошенный ресурс не найден

NotFoundObjectResult: производный от ObjectResult. Также возвращает клиенту ответ в виде статусного кода HTTP 404 с дополнительной информацией

BadRequestResult: производный от StatusCodeResult. Возвращает статусный код 400, тем самым указывая, что запрос некорректен

BadRequestObjectResult: производный от ObjectResult. Возвращает статусный код 400 с некоторой дополнительной информацией

OkResult: производный от StatusCodeResult. Возвращает статусный код 200, который уведомляет об успешном выполнении запроса

OkObjectResult: производный от ObjectResult. Возвращает статусный код 200 с некоторой дополнительной информацией

CreatedResult: возвращает статусный код 201, который уведомляет о создании нового ресурса. В качестве параметра принимает адрес нового ресурса

CreatedAtActionResult: возвращает статусный код 201, который уведомляет о создании нового ресурса. В качестве параметра принимает название метода и контроллера, а также параметров запроса, которые вместе создают адрес нового ресурса

CreatedAtRouteResult: возвращает статусный код 201, который уведомляет о создании нового ресурса. В качестве параметра принимает название маршрута, который используется для создания адреса нового ресурса

AcceptedResult: возвращает статусный код 202

AcceptedAtActionResult: возвращает статусный код 202. В качестве параметра принимает название метода и контроллера, а также параметров запроса

AcceptedAtRouteResult: возвращает статусный код 202. В качестве параметра принимает название и параметры маршрута

ChallengeResult: используется для проверки аутентификации пользователя

JsonResult: возвращает в качестве ответа объект или набор объектов в формате JSON

PartialViewResult: производит рендеринг частичного представления в выходной поток

RedirectResult: перенаправляет пользователя по другому адресу URL, возвращая статусный код 302 для временной переадресации или код 301 для постоянной переадресации зависимости от того, установлен ли флаг Permanent.

RedirectToRouteResult: класс работает подобно RedirectResult, но перенаправляет пользователя по определенному адресу URL, указанному через параметры маршрута

RedirectToActionResult: выполняет переадресацию на определенный метод контроллера

<
RedirectToPageResult: выполняет переадресацию на определенную странцу Razor (относится к подсистеме RazorPages)

li>
LocalRedirectResult: перенаправляет пользователя по определенному адресу URL в рамках веб-приложения

ViewComponentResult: возвращает в ответ сущность ViewComponent

ViewResult: производит рендеринг представления и отправляет результаты рендеринга в виде html-страницы клиенту


## Переадресация

```asp
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

## Отправка статусных кодов

StatusCodeResult позволяет отправить любой статусный код клиенту:

```asp
public IActionResult Index()
{
    return StatusCode(401);
}
```

Подобным образом мы можем послать браузеру любой другой статусный код. Но для отдельных кодов статуса предназначены свои отдельные классы.

**HttpNotFoundResult** и **HttpNotFoundObjectResult**
NotFoundResult и NotFoundObjectResult посылает код 404, уведомляя браузер о том, что ресурс не найден. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.

Объекты обоих классов создаются методом NotFound. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию. Например, используем NotFoundObjectResult:

```asp
public IActionResult Index()
{
    return NotFound("Ресурс в приложении не найден");
}
```

UnauthorizedResult и UnauthorizedObjectResult
UnauthorizedResult посылает код 401, уведомляя пользователя, что он не автризован для доступа к ресурсу:

```asp
public IActionResult Index(int age)
{
    if (age < 18)
        return Unauthorized();
    return Content("Проверка пройдена");
}
```

Для создания ответа используется метод Unauthorized().

UnauthorizedObjectResult также посылает код 401, только позволяет добавить в ответ некоторый объект с информацией об ошибке:

```asp
public class HomeController : Controller
{
    public IActionResult Index(int age)
    {
        if (age < 18)
            return Unauthorized(new Error { Message = "параметр age содержит недействительное значение" });
        return Content("Проверка пройдена");
    }
}
class Error
{
    public string Message { get; set; }
}
```

BadResult и BadObjectResult
BadResult и BadObjectResult посылают код 400, что говорит о том, что запрос некорректный. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.

Эти классы можно применять, например, если в запросе нет каких-то параметров или данные представляют совсем не те типы, которые мы ожидаем получить, и т.д.

Объекты обоих классов создаются методом BadRequest. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию:

```asp
public IActionResult Index(string s)
{
    if(String.IsNullOrEmpty(s))
        return BadRequest("Не указаны параметры запроса");
    return View();
}
```

OkResult и OkObjectResult
OkResult и OkObjectResult посылают код 200, уведомляя об успешном выполнении запроса. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.

Объекты обоих классов создаются методом Ok(). Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию:

```asp
public IActionResult Index()
{
    return Ok("Запрос успешно выполнен");
}
```


## Отправка файлов
...
https://metanit.com/sharp/aspnet5/5.7.php


## Переопределение контроллеров

Как правило, для создания контроллера достаточно унаследовать свой класс от базового класса Controller. Однако если нам необходимо, чтобы наши контроллеры реализовали некоторую общую логику, мы можем определить свой базовый класс контроллера и уже от него наследовать остальные контроллеры. Либо мы также можем переопределить некоторые методы базового класса Controller, если они нас не устраивают.

Что мы можем в контроллере переопределить? В базовом классе Controller среди всех прочих методов есть три интересных метода:

Метод OnActionExecuting() выполняется при вызове метода контроллера до его непосредственного выполнения.

Метод OnActionExecuted() выполняется после выполнения метода контроллера.

Метод OnActionExecutionAsync() представляет асинхронную версию метода OnActionExecuting().

**Замечание**: это похоже на callback или фильтры перед действиями контроллеров в Rails

## Контекст контроллера
https://metanit.com/sharp/aspnet5/5.9.php

## Передача зависимотей в контроллер
https://metanit.com/sharp/aspnet5/5.10.php


## Представления Razor

Замечание: полноценной html-страницей представление все равно не является, потому что во время выполнения эти представления компилируются в сборки и уже затем используются для генерации html-страниц, которые видит пользователь в своем браузере.

Для хранения представлений в проекте ASP.NET MVC предназначена папка Views:

Представления в ASP.NET MVC Core
В этой папке уже есть некоторая подструктура. Во-первых, как правило, для каждого контроллера в проекте создается подкаталог в папке Views, который называется по имени контроллера и который хранит представления, используемые методами данного контроллера. Так, по умолчанию имеется контроллер HomeController и для него в папке Views есть подкаталог Home с представлениями для методов контроллера HomeController.

Также здесь есть папка Shared, которая хранит общие представления для всех контроллеров. По умолчанию это файлы _Layout.cshtml (используется в качестве мастер-страницы), Error.cshtml (использутся для отображения ошибок) и _ValidationScripsPartial.cshtml (частичное представление, которое подключает скрипты валидации формы).

И в корне каталога Views также можно найти два файла _ViewImports.cshtml и _ViewStart.cshtml. Эти файлы содержат код, который автоматически добавляется ко всем представлениям. _ViewImports.cshtml устанавливает некоторые общие для всех представлений пространства имен, а _ViewStart.cshtml устанавливает общую мастер-страницу.


ViewResult
За работу с представлениями отвечает объект ViewResult. Он производит рендеринг представления в веб-страницу и возвращает ее в виде ответа клиенту.

Чтобы возвратить объект ViewResult используется метод View:

```asp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```


Вызов метода View возвращает объект ViewResult. Затем уже ViewResult производит рендеринг определенного представления в ответ. По умолчанию контроллер производит поиск представления в проекте по следующим путям:

/Views/Имя_контроллера/Имя_представления.cshtml
/Views/Shared/Имя_представления.cshtml
Согласно настройкам по умолчанию, если название представления не указано явным образом, то в качестве представления будет использоваться то, имя которого совпадает с именем действия контроллера.Например, вышеопределенное действие Index по умолчанию будет производить поиск представления Index.cshtml в папке /Views/Home/.

Метод View() имеет четыре перегруженных версии:

View(): для генерации ответа используется представление, которое по имени совпадает с вызывающим методом

View(string viewName): в метод передается имя представления, что позволяет переопределить используемое по умолчанию представление

View(object model): передает в представление данные в виде объекта model

View(string viewName, object model): переопределяет имя представления и передает в него данные в виде объекта model

Вторая версия метода позволяет переопределить используемое представление. Если представление находится в той же папке, которая предназначена для данного контроллера, то в метод View() достаточно передать название представления без расширения:

```asp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View("About");
    }
}
```
В этом случае метод Index будет использовать представление Views/Home/About.cshtml. Если же представление находится в другой папке, то нам надо передать полный путь к представлению:

```asp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View("~/Views/Some/Index.cshtml");
    }
}
```

Представление в ASP.NET MVC может содержать не только стандартный код html, но и также вставки кода на языке C#. Для обработки кода, содержащего как элементы html, так и конструкции C#, используется движок представлений.

В действительности при вызове метода View контроллер не производит рендеринг представления и не генерирует разметку html. Контроллер только готовит данные и выбирает, какое представление надо возвратить в качестве объекта ViewResult. Затем уже объект ViewResult обращается к движку представления для рендеринга представления в выходной ответ.

По умолчанию в ASP.NET MVC Core используется один движок представлений - Razor. Хотя при желании мы можем также использовать какие-то другие сторонние движки или создать свой движок представлений самостоятельно.

Весь код в пределах блока расценивается как код c#. Однако с помощью конструкции @: мы можем в блоке кода выводить на веб-страницу текст:

```asp
@{ 
    string head = "Hello world";
    @: <b>Привет мир!</b>
    head = head + "!!";
}
<p>@head</p>
```

## Передача данных в представление

Существуют различные способы передачи данных из контроллера в представление:

ViewData

ViewBag

Модель представления

ViewData
ViewData представляет словарь из пар ключ-значение:

```asp
public IActionResult Index()
{
    ViewData["Message"] = "Hello ASP.NET Core";
 
    return View();
}
```

Здесь динамически определяется во ViewData объект с ключом "Message" и значением "Hello ASP.NET Core". При этом в качестве значения может выступать любой объект. И после этому мы можем его использовать в представлении:

```asp
@{
    ViewData["Title"] = "Index";
}
<h2>@ViewData["Title"].</h2>
<h3>@ViewData["Message"]</h3>
 
<p>Random text.</p>
```

Причем не обязательно устанавливать все объекты во ViewData в контроллере. Так, в данном случае объект с ключом "Title" устанавливается непосредственно в представлении.

ViewBag
ViewBag во многом подобен ViewData. Он позволяет определить различные свойства и присвоить им любое значение. Так, мы могли бы переписать предыдущий пример следующим образом:

```asp
public IActionResult Index()
{
    ViewBag.Message = "Hello ASP.NET Core";
 
    return View();
}
```

И таким же образом нам надо было бы изменить представление:

```asp
@{
    ViewBag.Title = "Index";
}
<h2>@ViewBag.Title.</h2>
<h3>@ViewBag.Message</h3>
 ```
 
 
<p>Random text.</p>
И не важно, что изначально объект ViewBag не содержит никакого свойства Message, оно определяется динамически. При этом свойства ViewBag могут содержать не только простые объекты типа string или int, но и сложные данные. Например, передадим список:

```asp
public IActionResult Index()
{
    ViewBag.Countries = new List<string> { "Бразилия", "Аргентина", "Уругвай", "Чили" };
    return View();
}
```

И также в представлении мы можем получить этот список:

```asp
<h3>В списке @ViewBag.Countries.Count элемента:</h3>
 
@foreach(string country in ViewBag.Countries)
{
    <p>@country</p>
}
```
    
Правда, чтобы выполнять различные операции, может потребоваться приведение типов, как в данном случае.

Модель представления
Модель представления является во многих случаях более предпочтительным способом для передачи данных в представление. Для передачи данных в представление используется одна из версий метода View:

```asp
public IActionResult Index()
{
    List<string> countries = new List<string> { "Бразилия", "Аргентина", "Уругвай", "Чили" };
    return View(countries);
}
```

В метод View передается список, поэтому моделью представления Index.cshtml будет тип List<string> (либо IEnumerable<string>). И теперь в представлении мы можем написать так:

```asp
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
    
В самом начале представления с помощью директивы @model устанавливается модель представления. Тип модели должен совпадать с типом объекта, который передается в метод View() в контроллере.

Установка модели указывает, что объект Model теперь будет представлять объект List<string> или список. И мы сможем использовать Model в качестве списка.

Модель представления в ASP.NET MVC Core
Представления, для которых определена модель, еще называют строго типизированными или strongly-typed views.

## Мастер страница

В каждом представлении через синтаксис Razor доступно свойство Layout, которое хранит ссылку на мастер-страницу. Здесь в качестве мастер страницы устанавливается файл _Layout.cshtml. При этом расширение можно не использовать.

Когда будет происходить рендеринг представления, то система будет искать мастер страницу _Layout по следующим путям:

/Views/[Название_контроллера]/_Layout.cshtml
/Views/Shared/_Layout.cshtml
Если в обеих папках: и в /Views/[Название_контроллера], и в /Views/Shared/ имеется файл с одинаковым именем, например, _Layout.cshtml, то к представлению применяется файл, который находится с ним в одной папке как более приоритетный. То есть таким образом мы можем определить для представлений каждого отдельного контроллера или представлений, которые находятся в одной папке, свою отдельную мастер-страницу.

Если вдруг мы хотим глобально по всему проекту поменять мастер-страницу на другой файл, который расположен в какой-то другой папке, например, в корне каталога Views, то нам надо использовать полный путь к файлу:

```asp
@{
    Layout = "~/Views/_Layout.cshtml";
}
```

## Секции

Кроме метода RenderBody(), который вставляет освновное содержимое представлений, мастер-страниц может также использовать специальный метод RenderSection() для вставки секций. Мастер-страница может иметь несколько секций, куда представления могут поместить свое содержимое. Например, добавим к мастер-странице _Master.cshtml секцию footer:

```asp
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

    
```asp
@{
    ViewData["Title"] = "Home Page";
    Layout = "~/Views/_Master.cshtml";
}
<h2>Представление Index.cshtml</h2>
 
@section Footer {
    Все права защищены. Site Corp. 2016.
}
```    
    
Но при таком подходе, если у нас есть куча представлений, и мы вдруг захотели определить новую секцию на мастер-странице, нам придется изменить все имеющиеся представления, что не очень удобно. В этом случае мы можем воспользоваться одним из вариантов гибкой настройки секций.

Первый вариант заключается в использовании перегруженной версии метода RenderSection, которая позволяет указать, что данную секцию не обязательно определять в представлении. Чтобы отметить секцию Footer в качестве необязательной, надо передать в метод в качестве второго параметра значение false:

1
<footer>@RenderSection("Footer", false)</footer>
Второй вариант позволяет задать содержание секции по умолчанию, если данная секция не определена в представлении:

```asp
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
    
## Частитчные представления
В приложениях на ASP.NET MVC кроме обычных представлений и мастер-страниц можно также использовать частичные представления или partial views. Их отличительной особенностью является то, что их можно встраивать в другие обычные представления. Частичные представления могут использоваться также как и обычные, однако наиболее удобной областью их использования является рендеринг результатов AJAX-запроса. По своему действию частичные представления похожи на секции, которые использовались в прошлой теме, только их код выносится в отдельные файлы.

Частичные представления полезны для создания различных панелей веб-страницы, например, панели меню, блока входа на сайт, каких-то других блоков.

За рендеринг частичных представлений отвечает объект PartialViewResult, который возвращается методом PartialView. Итак, определим в контроллере HomeController новое действие GetMessage:

```asp
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

```as[
<h3>Частичное представление</h3>
```
Теперь обратимся к методу GetMessage как к обычному действию контроллера, и оно нам вернет частичное представление:

Частичные представления в ASP.NET MVC Core
По своему действию частичное представление похоже на обычное, только для него по умолчанию не определяется мастер-страница.

Встраивание частичного представления в обычное
Теперь рассмотрим, как мы можем встраивать частичные представления в обычные. Для этого изменим представление Index.cshtml:

```asp
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

```asp
@{await Html.RenderPartialAsync("_GetMessage");}
```
    Html.RenderPartialAsync напрямую пишет вывод в выходной поток в асинхронном режиме, поэтому может работать чуть быстрее, чем Html.PartialAsync.

Одна из перегруженных версий методов Html.PartialAsync / Html.RenderPartialAsync позволяет передать модель в частичное представление. В итоге у нас получится стандартное строго типизированное представление. Например, в качестве второго параметра список строк:

1
@await Html.PartialAsync("_GetMessage", new List<string> { "Lumia 950", "iPhone 6S", "Samsung Galaxy s 6", "LG G 4" })
Поскольку здесь в частичное представление передается список строк, то мы можем использовать модель List<string>, чтобы получить этот список. Теперь изменим частичное представление _GetMessage.cshtml:

```asp
@model IEnumerable<string>
<h3>Список моделей</h3>
<ul>
    @foreach(string s in Model)
    {
        <li>@s</li>
    }
</ul>
```

## Внедрение зависимостей в представление
    
    
## Работа с формами
https://metanit.com/sharp/aspnet5/7.8.php
    

## Маршрутизация 
    https://metanit.com/sharp/aspnet5/11.5.php

## Маршрутизация на основе аттрибутов

Фреймворк MVC позволяет использовать в приложении маршрутизацию на основе атрибутов. Такой тип маршрутизации еще называется Attribute-Based Routing. Атрибуты предоставляют более гибкий способ определения маршрутов. Маршруты, определенные с помощью атрибутов, имеют приоритет по сравнению с маршрутами, определенными в классе Startup.

Маршрутизация на основе атрибутов может применяться независимо от того, какой компонент отвечает в приложении за маршрутизацию - RouterMiddleware или EndpointRoitingMiddleware+EndpointMiddleware.

Для определения маршрута, необходимо использовать атрибут [Route]:

```asp
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

```asp
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
    
```asp
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
    
```asp
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

```asp
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

Атрибуты маршрутизации в ASP.NET MVC Core
Определение маршрутов с помощью атрибутов подчиняется тем же правилам, что определение маршрутов в классе Startup. Например, используем параметры и ограничения:

```asp
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

```asp
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

Использование префиксов
Допустим, у нас есть несколько методов, для которых определен свой маршрут, и мы хотим, чтобы эти маршруты начинались с одного определенного префикса. Например:

```asp
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

    
```asp
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
    
Параметры controller и action
От всех параметров шаблона маршрутов в атрибутах отличаются два параметра controller и action, которые ссылаются соответственно на контроллер и его действие. При использовании их надо помещать в квадратные скобки, а не в фигурные, как другие параметры:

```asp
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

```asp
[Route("[controller]")]
[Route("[action]")]
[Route("[controller]/[action]/{id?}")]
```
    Множественные маршруты
С помощью атрибутов можно задать несколько маршрутов для одного метода. Например:

```asp
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
    
```asp
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
    
    
   
Форма
```asp
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


## Конфигурационный файл для пустого проекта для подключения MVC

```ASP

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

var app = builder.Build();


// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}


//app.MapGet("/", () => "Hello World!");

app.UseDeveloperExceptionPage(); // Ошибки при отладке
app.UseHttpsRedirection();
app.UseStaticFiles(); // доставка статического содержимого

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();


```
## Null условнаяя проверка с объединением
```asp

string name = р?.Name ?? "No name"; // null условная операция с объединением

```

## Автоматические реализуемые свойства

```asp
puЫic string Nаш.е { get; set; } 

```
## Сопоставление с образцом

```asp

if (data is decimal d) { }

```
Замечание: сопоставлнеие с образцом также работает с оператором ```switch```

---

## Применение лямбда выражений

```asp
pubclic ViewResult Index() {
return View()
}

```asp
pubclic ViewResult Index() => View()
}

```

## Передача данных из контроллера в представление через ViewBag

В контроллере метода можно прописать
``` ViewBag.AnyProperty = 2;```

В представлении можно:

```
@ViewBag.AnyProperty
```
## Возможности механизма представления Razor

Можно добавлять значения в аттрибуты элементов разметки
Условные операторы if, switch
Циклы foreach


## Подключение Bootstrap и js

wwwroot папка
В ней должны быть ресурсы

В шаблоне сделать ссылку
```html
<link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.css"/>
<script src = "js/"></script>
```

## Настройка Postman для localhost

Finally fixed it by turning off File > Settings > General > SSL Certificate Verification


## Bundler and Minifier
Установить расширение



## Надо потом изучить
интерфейсы в С# тестирование моделей, контроллеров, представлений через xUnit


## Порядок работы над новым проектом
1. Создать пустой проект ASP.NET Core 6

2. Создать папки Models,Controllers,Views

3. Скопировать конфигурационный конфиг по MVC паттерн в Program.cs

4. В папке Views создадим импортируемый шаблон для пространства имен _ViewImports.cshtml
```asp
@using SportsStore.Models // название приложения ? проверить 
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers 

```

5. Создать проект модульного тестирования xUnit

6. Добавить в ссылки проект и Moq. Это можно сделать через файл конфигурации ```asp <PackageReference Include="Moq" Version="4.7.99" /> ``` или через NuGet

7. Создаем модель предметной области. В папке Models создаем файл Product.cs

8. Создаем миграции с помощью консольных команд

```asp
dotnet ef migrations add Itinial
dotnet ef database update

```

## Форма

```asp

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
                <button class ="btn btn-primary" type="submit">Отправить</button>

            </form>

            </div>


            <p> <a href="/">Перейти на главную форму</a>.</p>
            
            <br>
            
            <a asp-action = "Index">Перейти на главную форму средствами asp</a>



```

## Get и Post запросы

```asp

// действие формы
        [HttpGet]
        public ViewResult RsvpForm()
        {
            //int hour = DateTime.Now.Hour;
            //ViewBag.Hour = hour;
            return View("RsvpForm");
        }


        [HttpPost]
        public ViewResult RsvpForm(GuestResponse guestResponse)
        // сохранить ответ
        {

            if (ModelState.IsValid)
            {
                Repository.AddResponse(guestResponse);
                return View("Thanks", guestResponse);
            }
            else
            {
                //Обнаружена ошибка проверки достоверности.
                return View();
            }
            //int hour = DateTime.Now.Hour;
            //ViewBag.Hour = hour;
            //return View();
        }

```

## Области. надо потестить, может быть полезны?
 
    https://metanit.com/sharp/aspnet5/11.9.php

## Модели
    https://metanit.com/sharp/aspnet5/8.1.php
## Html-хэлперы
    https://metanit.com/sharp/aspnet5/9.1.php
## Tag-хэлперы
    https://metanit.com/sharp/aspnet5/10.1.php
## Компоненты
    https://metanit.com/sharp/aspnet5/7.6.php
## Валидации моделей
    https://metanit.com/sharp/aspnet5/19.1.php

## Работа с Entity Framework. Сортировка, фильтрация, пагинация
    https://metanit.com/sharp/aspnet5/12.1.php

## Razor Pages - то же самое, что и MVC, только почти без контроллеров в чистом виде
    https://metanit.com/sharp/aspnet5/29.1.php

## Web API
    https://metanit.com/sharp/aspnet5/23.1.php
## Фильтры как в Rails
    https://metanit.com/sharp/aspnet5/18.1.php
## Применение аутентификации и авторизации средаствами ASP .NET Core
    https://metanit.com/sharp/aspnet5/15.1.php
    https://metanit.com/sharp/aspnet5/16.1.php

## Клиентская разработка
    https://metanit.com/sharp/aspnet5/13.6.php

## Кеширование
    https://metanit.com/sharp/aspnet5/14.1.php

## Сервер и публикация приложения
    https://metanit.com/sharp/aspnet5/2.7.php
    
## URL Rewriting
    https://metanit.com/sharp/aspnet5/24.1.php

## Глобализация и локализация
    https://metanit.com/sharp/aspnet5/28.1.php

## SignalR Core
    https://metanit.com/sharp/aspnet5/30.1.php
 
## Кроссдоменные запросы. CORS и кросс-доменные запросы
    https://metanit.com/sharp/aspnet5/31.1.php

## Тестирование
    https://metanit.com/sharp/aspnet5/22.1.php

## Dapper - альтернатива EF
    https://metanit.com/sharp/aspnet5/26.1.php

## Mongo DB
    https://metanit.com/sharp/aspnet5/27.1.php
    
## React.JS
    https://metanit.com/sharp/aspnet5/25.1.php

## Сервисы gRPC
    https://metanit.com/sharp/aspnet5/32.1.php
    
## Отправка email в ASP.NET Core
    https://metanit.com/sharp/aspnet5/21.1.php

## Загрузка файлов на сервер
    https://metanit.com/sharp/aspnet5/21.3.php
    
