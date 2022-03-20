## ASP.Net Core 6 ( > 2016 )

### Виды приложений на ASP

Можно создать пустое приложение
Можно создать веб-приложение
Можно создать веб-приложение MVC
Можно создать API веб-приложение


1. Создали пустой проект Asp Net Core
2. Создали папки Models,Controllers,Views
3. Создали класс модель в Models
4. Создать класс контроллера в Controllers. Нужно подключение using Microsoft.AspNetCore.Mvc 

```asp

**using Microsoft.AspNetCore.Mvc**;

namespace Example.Controllers;

    public class HomeController : Controller
    {

        public IActionResult Index()
        {
        return View();
        }

    }

```
Замечание: можно создать отдельный файл класса с глобальными подключениями ```global using Microsoft.AspNetCore.Mvc; ```

5. Создадим представление пустое Razor. Это файл с расширением cshtml в папке View. В представлении подключаем модель. Далее можно использовать переданный объект модели из action контроллера в представлении через @Model

```asp

@model Example.Models.MyModel;
<h1>@Model.Message</h1>

```
6. Настройка работы под паттерн MVC в файле Program.cs



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

7.
После определения моделей надо выбрать хранилище данных для этих моделей. Мы будем использовать MS SQL Server. Для работы с MS SQL Server компания Microsoft рекомендует использовать ORM-технологию Entity Framework, хотя ее использование необязательно. Мы также можем применять другие ORM-технологии или доступные средства ADO.NET. Преимущество фреймворка Entity Framework состоит в том, что он позволяет абстрагироваться от структуры конкретной базы данных и вести все операции с данными через модель.

8. Далее надо через менеджер пакетов Nuget установить Entity Framework Sql Server 

9.
Чтобы взаимодействовать с базой данных нам нужен контекст данных. Причем Entity Framework Core использует подход Code First, при котором нам надо сначала определить модели и контекст данных, а потом уже исходя и этих моделей и класса контекста будет создаваться бд и все ее таблицы.

MobileContext.cs в Models
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

10.

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

11. Настройка подключения в файле Program.cs

```asp
// Подключение базы данных SQL Server
string connection = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<MobileContext>(options => options.UseSqlServer(connection));
```

12. После того, как мы настроили подключение к базе данных мы можем работать через модели посредством контекста с базой данных
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


13. Представление получает доступ к данным, которые были переданы из метода контроллера

```asp
@model IEnumerable<Example.Models.Phone>
```
Первая строка устанавливает модель представления - та сущность, которая будет доступна в представлении через объект Model. В данном случае это объект IEnumerable<MobileStore.Models.Phone>, так как в контроллере в методе Index мы передаем список смартфонов, а список - объект List представляет интерфейс IEnumerable.

Теперь данные доступны посредством объекта @Model

**Замечание**: Каждый метод контроллера использует по умолчанию одноименный метод пердставления

14 

```asp
@{
    Layout = null
}

Далее идет блок кода, в котором выражение Layout = null указывает, что мастер-страница (макет) не будет применяться к этому представлению

```

15. Применение мастер-страницы (макет). Файл _Layout.cshtml должен находиться в папке Views/Shared
```asp
@{
    Layout ="_Layout"; 
}
```

16.
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

17. Можно применять аттурибуты NonController, ActionName и NonAction


18. Передача данных в контроллер
Вместе с запросом приложению могут приходить различные данные. И чтобы получить эти данные, мы можем использовать разные способы. Самым распространенным способом считается применение параметров.

Определение в методах контроллера параметров ничем не отличается от определения параметров в языке C#. Параметры могут представлять примитивные типы, как int или string, а могут представлять и более сложные классы.

Передавать значения для параметров можно различными способами. При отправке GET-запроса значения передаются через строку запроса. Стандартный get-запрос принимает примерно следующую форму: название_ресурса?параметр1=значение1&параметр2=значение2. Например, определим следующий метод:

Система привязки MVC, которую мы позже рассмотрим, сопоставляет параметры запроса и параметры метода по имени. То есть, если в строке запроса идет параметр a, то его значение будет передаваться именно параметру метода, который также называется a. При этом должно быть также соответствие по типу, то есть если параметр метода принимает числовое значение, то и через строку запроса надо передавать для этого параметра число, а не строку.

Post запросы
Чтобы система могла связать параметры метода и данные формы, необходимо, чтобы атрибуты name у полей формы соответствовали названиям параметров.

19. Получение данных из контекста запроса

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


20 Результаты действий action

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


21. Переадресация

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

22. # Отправка статусных кодов

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

Все почти также как в ```Rails```

- Bootstrap идет из коробки
- Подключение css и javascript удобное
- Есть также свои дескрипторы для тегов
- Есть валидации в моделях
- Аналог params, который надо изучить
- Есть соглашения по рендерингу
- Роутинг еще не смотрел
- Тестирвоание через TDD еще не смотрел
- Настройку конфигурации еще не смотрел
- Подключение Entity Framework еще не смотрел
- Есть частичные шаблоны
- Надо посмотреть сесиии
- Развертывание как делать

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
интерфейсы в С#
тестирование моделей, контроллеров, представлений через xUnit



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












