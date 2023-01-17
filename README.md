# ASP.Net Core

## Версия

### ASP.Net Core 6
### EntityFrameworkCore (6.0.7)

---
# План работы

1. Написать CRUD c базой данных SQL Server и на фронт bootstrap
  
  1.1 Применить скаффолд
   
  1.2 Применить миграции
  
2. Развернуть на VPS Ubuntu

3. Развернуть на VPS через docker

4. Настроить CI/CD c тестированием

# Обновление

1. Добавить базу данных Postgre, MySQL, SQLite, удаленную базу данных в облаке, Redis, Mongodb

1.1. Применить механизм генерации тестовых данных

2. Настроить связи межуд таблицами. Минимум 5 штук

3. В репозитории проекта описать UML

4. В архитектуру проекта добавить интерфейсы

5. На VPS применить домен и SSL сертификат

6. Применить задачи RabbitMQ

7. Применить кеширование

8. Подключить новый API к проекту

9. Фронт React, Angular, Vue

10. Сделать микросервисную архитектуру

11. Применить kubernetes

12. Тестирование API

13. Автоматизированное тестирование Selenium

14. Написание ручных тестов UI


## Program.cs

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

---

### appsettings.json

По умолчанию у нас база данных отсутствуют. Поэтому в конструктор MobileContext определен вызов Database.EnsureCreated(), который при отсутствии базы данных автоматически создает ее. Если база данных уже есть, то ничего не происходит.
Чтобы подключаться к базе данных, нам надо задать параметры подключения. Для этого изменим файл appsettings.json. По умолчанию он содержит только настройки логгирования:

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


### Команды

```dotnet new mvc --output TestMVC```

```dotnet tool install -g dotnet-aspnet-codegenerator```

```dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design```

```dotnet add package Microsoft.EntityFrameworkCore.Design```

```dotnet tool install --global dotnet-ef```

```dotnet add package Microsoft.EntityFrameworkCore.SQLite```

```dotnet add package Microsoft.EntityFrameworkCore.SqlServer```

```dotnet aspnet-codegenerator controller --controllerName Home --model Beer --dataContext BeerContext --useDefaultLayout -outDir Controllers -f --useSqlite```

```dotnet ef database drop --force```

```dotnet ef migrations add InitialCreate```

**Замечание:** возможно надо перезапустить Visual Studio

**Замечание**: --project Name, если несколько проектов. Помогает также очистка и пересборка решения.

```dotnet ef database update```

**Замечание**: --connection "Data Source=My.db"


---
## Context for SQLite

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


---







---

## Модель
---
## Валидация модели на стороне сервера

Важную роль в ASP.NET Core MVC играет валидация входных данных. Валидация позволяет проверить входные данные на наличие неправильных, корректных значений и должным образом обработать эти значения. Валидация модели в ASP.NET Core MVC базируется на общем механизме валидации, который имеется в .NET, однако ASP.NET Core MVC добавляет некоторую дополнительную инфраструктуру, которая облегчает процесс валидации.

---

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

Валидация на стороне сервера, то есть в контроллере, осуществляется посредством помощью проверки свойства ModelState.IsValid. Объект ModelState сохраняет все значения, которые пользователь ввел для свойств модели, а также все ошибки, связанные с каждым свойством и с моделью в целом. Если в объекте ModelState имеются какие-нибудь ошибки, то свойство ModelState.IsValid возвратит false.


```Csharp

            if (ModelState.IsValid)
                return Content($"{person.Name} - {person.Age}");

            string errorMessages = "";
            // проходим по всем элементам в ModelState
            foreach (var item in ModelState)
            {
                // если для определенного элемента имеются ошибки
                if (item.Value.ValidationState == ModelValidationState.Invalid)
                {
                    errorMessages = $"{errorMessages}\nОшибки для свойства {item.Key}:\n";
                    // пробегаемся по всем ошибкам
                    foreach (var error in item.Value.Errors)
                    {
                        errorMessages = $"{errorMessages}{error.ErrorMessage}\n";
                    }
                }
            }
            return  View(person); // errorMessages;
            
```

## Добавление собственных валидаций 

При необходимости мы сами можем валидировать значения и добавлять в ModelState информацию об ошибках. Для этого применяется метод ModelState.AddModelError.

```Csharp

 if (person.Name == "admin" && person.Age == 25)
            {
                // добавляем ошибку уровня модели
                ModelState.AddModelError("", "Некорректные данные");
            }

            if (person.Age > 110 || person.Age < 0)
            {
                ModelState.AddModelError("Age", "Возраст должен находиться в диапазоне от 0 до 110");
            }
            if (person.Name?.Length < 3)
            {
                ModelState.AddModelError("Name", "Недопустимая длина строки. Имя должно иметь больше 2 символов");
            }

```

## Валидация на стороне клиента

Кроме валидации на стороне сервера с помощью свойства ModelState в ASP.NET Core MVC можно применять валидацию на стороне клиента, то есть на веб-странице. Валидация на стороне клиента позволяет уменьшить количество обращений к серверу и произвести все действия по проверке значений непосредственно при вводе данных.

### Подключение скирптов

```html
<script src="https://ajax.aspnetcdn.com/ajax/jQuery/jquery-3.5.1.min.js"></script>
<script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.17.0/jquery.validate.min.js"></script>
<script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.10/jquery.validate.unobtrusive.min.js"></script>
```
Для применения скриптов в разметке должен быть tag-helper:

```Csharp
 <span asp-validation-for="Name" class="text-danger"></span>
```

Таким образом, валидация на стороне клиента позволит уменьшить нагрузку на сервер. Тем не менее стоит учитывать, что на стороне клиента по тем или иным причинам может быть отключен javascript, либо данные отправляются в среде, где эти скрипты не применяются. Соответственно валидация на стороне сервера по прежнему актуальна. Кроме того, на стороне сервера могут проводиться дополнительные проверки, которые не могут быть выполнены на стороне клиента.

## Аттрибуты валидации

Замечание: для свойств тех типов данных, которые не могут принимать значение null, нам в любом случае надо предоставить значение, как в случае со свойством Age. Поэтому для него использовать атрибут Required необязательно.


## Аттрибут проверки регулярного выражения

```Csharp
[RegularExpression(@"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}", ErrorMessage = "Некорректный адрес")]
public string? Email { get; set; }
```

```Csharp
[EmailAddress (ErrorMessage = "Некорректный адрес")]
public string? Email { get; set; }
```


## Атрибут StringLength

Чтобы пользователь не мог ввести очень длинный текст, применяется атрибут StringLength. Первым параметром в конструкторе атрибута идет максимальная допустимая длина строки. Именованные параметры, в частности MinimumLength и ErrorMessage, позволяют задать дополнительные опции отображения.

```Csharp
[StringLength(50, MinimumLength = 3, ErrorMessage = "Длина строки должна быть от 3 до 50 символов")]
    public string? Name { get; set; }
```

## Атрибут Range

Атрибут Range определяет минимальные и максимальные значения для числовых данных.

```Csharp
[Range(1, 110, ErrorMessage = "Недопустимый возраст")]
public int Age { get; set; }
```

## Атрибут Compare

Атрибут Compare гарантирует, что два свойства объекта модели имеют одно и то же значение. Если, например, надо, чтобы пользователь ввел пароль дважды:

```Csharp
[Required]
public string? Password { get; set; }
 
[Compare("Password", ErrorMessage = "Пароли не совпадают")]
public string? PasswordConfirm { get; set; }
```

## Атрибут Remote

Атрибут Remote из пространства имен Microsoft.AspNetCore.Mvc; для валидации свойства выполняет запрос на сервер к определенному методу контроллера. И если требуемый метод контроллера вернет значение false, то валидация не пройдена. Например:

```Csharp
[Remote(action: "CheckEmail", controller: "Home", ErrorMessage ="Email уже используется")]
public string Email { get; set; }
``

## ValidationSummaryTagHelper

ValidationSummaryTagHelper применяется для отображения сводки ошибок валидации. Он применяется к элементу <div> в виде атрибута asp-validation-summary:

```html
<div asp-validation-summary="ModelOnly"/>
```

В качестве значения атрибут asp-validation-summary принимает одно из значений перечисления ValidationSummary:

- None: ошибки валидации не отображаются

- ModelOnly: отображаются только ошибка валидации уровня модели, ошибки валидации для отдельных свойств не отображаются

- All: отображаются все ошибки валидации


## Стилизация валидаций

Когда происходит валидация, то при отображении ошибок соответствующим полям присваиваются определенные классы css:

- для блока ошибок, который генерируется хелпером ValidationSummaryTagHelper, при наличии ошибок устанавливается класс validation-summary-errors. Если ошибок нет, то данный блок не отображается

- для элемента <span>, который отображает ошибку для каждого отдельного поля и который генерируется хелпером ValidationTagHelper, при наличии ошибок устанавливается класс field-validation-error. Если ошибок нет, то данный элемент имеет класс field-validation-valid

- для поля ввода при наличии ошибок устанавливается класс field-validation-error. Если ошибок нет, то устанавливается класс valid


```html
<style>
.field-validation-error {
    color: #b94a48;
}
  
input.input-validation-error {
    border: 1px solid #b94a48;
}
input.valid {
    border: 1px solid #16a085;
}
 
.validation-summary-errors {
    color: #b94a48;
}
</style>

```

## Создание атрибута валидации
    
Встроенные атрибуты валидации охватывают большую часть возможных ситуаций, тем не менее этих атрибутов может оказаться недостаточно, особенно в каких-то специфических сценариях. Однако .NET позволяет определять собственные атрибуты валидации со своей логикой.
    
Для создания атрибута валидации необходимо унаследовать класс атрибута от ValidationAttribute и переопределить его метод IsValid().    
    
```Csharp
public class PersonNameAttribute : ValidationAttribute
    {
        //массив для хранения допустимых имен
        string[] _names;
 
        public PersonNameAttribute(string[] names)
        {
            _names = names;
        }
        public override bool IsValid(object? value)
        {
            return value != null && _names.Contains(value);
        }
```
    
Чтобы применить логику валидации, надо переопределить метод IsValid, предоставленный базовым классом. Параметр этого метода как раз и представляет то значение, которое надо валидировать.    
    
```Csharp
[PersonName(new string[] {"Tom", "Sam", "Alice" }, ErrorMessage ="Недопустимое имя")]
```

## Самовалидация и IValidatableObject
    
 Самовалидация представляет собой процесс, при котором модель запускает механизм валидации из себя самой. И сама инкапсулирует всю логику валидации.

Для этого класс модели должен реализовать интерфейс IValidatableObject:

    
```Csharp
   using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
 
public class Person : IValidatableObject
{
    public string? Name { get; set; }
 
    public string? Email { get; set; }
    public string? Password { get; set; }
    public int Age { get; set; }
 
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var errors = new List<ValidationResult>();
 
        if (string.IsNullOrWhiteSpace(Name))
        {
            errors.Add(new ValidationResult("Введите имя!", new List<string>() { "Name" }));
        }
        if (string.IsNullOrWhiteSpace(Email))
        {
            errors.Add(new ValidationResult("Введите электронный адрес!"));
        }
        if (Age < 0 || Age > 120)
        {
            errors.Add(new ValidationResult("Недопустимый возраст!"));
        }
 
        return errors;
    }
} 
```
    
В данном случае нам надо реализовать метод Validate() и возвратить коллекцию объектов ValidationResult, которые и будут содержать все ошибки валидации. Если в конструктор ValidationResult передается только сообщение об ошибке, тогда данная ошибка будет относиться ко всей модели в целом. Однако с помощью второго параметра можно указать конкретный список свойств модели, к которым относится ошибка. То есть в данном случае ошибки для свойств Email и Age будут считаться ошибками для всей модели, а ошибка для Name будет ошибкой только этого конкретного свойства:    
    

## Аннотации данных
    
Кроме атрибутов валидации модели также могут иметь дополнительные атрибуты, которые называются аннотации данных и которые располагаются в пространстве имен System.ComponentModel.DataAnnotations. Эти атрибуты определяют различные правила для отображения свойств модели.
 
    
## Атрибут Display   
    
```Csharp
public class Person
    {
        [Display(Name="Имя и фамилия")]
        public string? Name { get; set; }
        [Display(Name = "Электронная почта")]
        public string? Email { get; set; }
        [Display(Name = "Домашняя страница")]
        public string? HomePage { get; set; }
        [Display(Name = "Пароль")]
        public string? Password { get; set; }
        [Display(Name = "Возраст")]
        public int Age { get; set; }
    }    
```

## DisplayFormat

Атрибут DisplayFormat позволяет задать формат отображения свойства. Например, пусть у нас будет в модели свойство по типу DateTime:
    
```Csharp
[Display(Name = "Дата рождения")]
[DisplayFormat(DataFormatString = "{0:dd.MM.yyyy}", ApplyFormatInEditMode = true)]
public DateTime DateOfBirth { get; set; } 
```

Атрибут DisplayFormat настраивает формат с помощью ряда свойств. Так, свойство DataFormatString указывает на сам формат. Например, в данном случае мы прописали в формате вывод дня, месяца и года даты рождения.

Свойство ApplyFormatInEditMode позволяет применять форматирование к свойству даже в режиме редактирования.   

## Фильтры
    
 Фильтры позволяют выполнять некоторые действия до или после определенной стадии обработки запроса. В ASP.NET Core MVC имеются следующие типы фильтров:   
    
  Фильтры авторизации: определяют, авторизован ли пользователь для выполнения текущего запроса. Если пользователь не авторизованн для доступа к ресурсу, то фильтр завершает обработку запроса.  
    
  Фильтры ресурсов: выполняются после фильтров авторизации. Его метод OnResourceExecuting() выполняется до всех остальных фильтров и до привязки модели, а его метод OnResourceExecuted() выполняется после всех остальных фильтров  
    
  Фильтры действий: применяется только к действиям контроллера, запускается после фильтра ресурсов как до, так и после выполнения метода контроллера  
    
  Фильтры исключений: определяют действия в отношении необработанных исключений  
    
   Фильтры результатов действий: фильтр применяется к результатам методов контроллера, выполняется как до, так и после получения результата 
    
    Вместе все эти типы фильтров образуют конвейер фильтров (filter pipeline), который встроен в процесс обработки запроса в MVC и который начинает выполняется после того, как инфраструктура MVC выбрала метод контроллера для обработки запроса. На разных этапах обработки запроса в этом конвейере вызывается соответствующий фильтр:
    
    
 ```Csharp
    
    using Microsoft.AspNetCore.Mvc.Filters;
 
public class SimpleActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // код метода
    }
 
    public void OnActionExecuted(ActionExecutedContext context)
    {
        // код метода
    }
}
 ```
 
Замечание: При реализации интерфейсов следует учитывать, что мы можем реализовать либо только синхронный, либо только асинхронный вариант. Если же класс будет реализовать оба варианта, то система будет вызывать только метод асинхронного интерфейса, а реализация синхронного интерфейса будет игнорироваться.
    
   
К примеру, создадим свой фильтр. Для этого добавим в проект папку Filters, в которой будут храниться фильтры. И далее добавим в эту папку новый класс SimpleResourceFilter, который будет представлять простейший фильтр ресурсов
    

```Csharp
    
using Microsoft.AspNetCore.Mvc.Filters;
 
namespace MvcApp.Filters
{
    public class SimpleResourceFilter : Attribute, IResourceFilter
    {
        public void OnResourceExecuting(ResourceExecutingContext context)
        {
            context.HttpContext.Response.Cookies.Append("LastVisit", DateTime.Now.ToString("dd/MM/yyyy HH-mm-ss"));
        }
 
        public void OnResourceExecuted(ResourceExecutedContext context)
        {
            // реализация отсутствует
        }
    }
    
```
    
Замечание: Также стоит отметить, что класс фильтра также наследует класс Attribute, так как фильтры реализуются в основном именно как атрибуты, что упрощает их применение.  

Применение: Таким образом, фильтры позволяют нам избежать повторения в соответствии принципом DRY (Don't Repeat Yourself)
    
 ```Csharp

    using Microsoft.AspNetCore.Mvc;
using MvcApp.Filters;  // пространство имен фильтра SimpleResourceFilter

    
namespace MvcApp.Controllers
{
    [SimpleResourceFilter]
    public class HomeController : Controller
    {
        public IActionResult Index() => View();
    }
}
 ```

    
 ## Область действия фильтров
    
 В ASP.NET Core MVC фильтры имеют три области действия:

  - Метод контроллера. Атрибут применяется к методу контроллера: 
  - Область действия контроллер. Атрибут применяется к классу контроллера:  
  - Глобальная область действия, при которой фильтр применяется ко всем методам всех контроллеров приложения.  
  
 Для определения фильтра как глобального надо при подключении соответствующих сервисов MVC добавить фильтр в коллекцию фильтров. Подключение глобально для всех сервисов MVC:
    
```Csharp
    
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMvc(options =>
{
    options.Filters.Add(typeof(SimpleResourceFilter)); // подключение по типу
 
    // альтернативный вариант подключения
    options.Filters.Add(new SimpleResourceFilter()); // подключение по объекту
    options.Filters.Add<SimpleResourceFilter>();  // применение типизированного метода

    }); 
```
    
 Установка отдельно для контроллеров и представлений:   
 
 ```Csharp
    
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(typeof(SimpleResourceFilter)); // подключение по типу
    options.Filters.Add(new SimpleResourceFilter()); // подключение по объекту
    options.Filters.Add<SimpleResourceFilter>(); 
});
    
 ```
 
Кроме того, следует отметить, что есть определенная очередность выполнения фильтров в зависимости от области действия:
    
 Очередность

Область действия

Метод фильтра

1

Глобальная

On[Stage]Executing

2

Контроллер

On[Stage]Executing

3

Действие контроллера

On[Stage]Executing

4

Действие контроллера

On[Stage]Executed

5

Контроллер

On[Stage]Executed

6

Глобальная

On[Stage]Executed   


## Переопределение порядка фильтров
 
ASP.NET Core MVC позволяет переопределить порядок выполнения фильтров. Для этого фильтр должен реализовать интерфейс IOrderedFilter, который определяет свойство Order - это свойство устанавливает порядок выполнения фильтра - чем оно меньше, тем раньше будет выполняться фильтр. Посмотрим на примере. Изменим класс ControllerResourceFilter следующим образом:
 
    
 ```Csharp
    
 namespace MvcApp.Filters
{
    public class ControllerResourceFilter : Attribute, IResourceFilter, IOrderedFilter
    {
        public ControllerResourceFilter(int order) => Order = order;
        public int Order { get; }
 
        public void OnResourceExecuting(ResourceExecutingContext context)
        {
            Console.WriteLine("ControllerResourceFilter.OnResourceExecuting");
        }
 
        public void OnResourceExecuted(ResourceExecutedContext context)
        {
            Console.WriteLine("ControllerResourceFilter.OnResourceExecuted");
        }
    }
}
    
 ```
 
Теперь класс ControllerResourceFilter через конструктор устанавливает свойство Order.

И также изменим код контроллера:
    
```Csharp
    
using Microsoft.AspNetCore.Mvc;
using MvcApp.Filters;  // пространство имен фильтров
 
namespace MvcApp.Controllers
{
    [ControllerResourceFilter(int.MinValue)]
    public class HomeController : Controller
    {
        [ActionResourceFilter]
        public IActionResult Index() => View();
    }
}
    
```
Значение int.MinValue, которое передается через конструктор класса ControllerResourceFilter свойству Order, фактически означает, что этот фильтр будет запускаться раньше всех. В чем мы можем убедиться, запустив проект:
    

## Передача параметров в фильтры
    
Фильтры могут принимать некоторые значения из вне. Для их приема мы можем определить в классе фильтра конструктор с соответствующими параметрами. Например:    
 
```Csharp
 
public class SimpleResourceFilter : Attribute, IResourceFilter
{
    int _id;
    string _token;
    public SimpleResourceFilter(int id, string token)
    {
        _id = id;
        _token = token;
    }
    public void OnResourceExecuted(ResourceExecutedContext context)
    {
    }
 
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        context.HttpContext.Response.Headers.Add("Id", _id.ToString());
        context.HttpContext.Response.Headers.Add("Token", _token);
    }
}
    
```
    
Здесь конструктор фильтра принимает два параметра. При применении фильтра мы можем передать значения для этих параметров:   
    
```Csharp
    
public class HomeController : Controller
{
    [SimpleResourceFilter(30, "8h6ell3o5wor5ld8")]
    public IActionResult Index()
    {
        return View();
    }
}
    
```
    
Если фильтр устанавливается глобально, то также при его установке необходимо передать ему нужные значения:    
    
```Csharp
    
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews(options=> options.Filters.Add(new SimpleResourceFilter(22, "jkrgbjkjgjghddw")));
    
```
 
## Фильтры и Dependency Injection
    

Как и любые классы в ASP.NET Core, фильтры также могут принимать зависимости, устанавливаемые при старте приложения. Однако, если фильтр применяется непосредственно к контроллеру или его методу, то установка зависимостей через встроенный механизм Dependency Injection не будет работать. Все потому, что при применении атрибутов все значения должны быть переданы напрямую в их конструктор. Чтобы обойти это ограничение, надо использовать один из классов ServiceFilterAttribute или TypeFilterAttribute.    
    
 Например, пусть класс фильтра получает зависимость ILoggerFactory, которая позволяет создать объект логгера для логгирования событий приложения:   
    
```Csharp

sing Microsoft.AspNetCore.Mvc.Filters;
 
public class SimpleResourceFilter : Attribute, IResourceFilter
{
    ILogger _logger;
    public SimpleResourceFilter(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger("SimpleResourceFilter");
    }
    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        _logger.LogInformation($"OnResourceExecuted - {DateTime.Now}");
    }
 
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        _logger.LogInformation($"OnResourceExecuting - {DateTime.Now}");
    }
}
    
```

 Класс TypeFilterAttribute создает объект фильтра с помощью фабрики Microsoft.Extensions.DependencyInjection.ObjectFactory, а с помощью механизма DI устанавливает все зависимости для создаваемого фильтра. Применение TypeFilterAttribute:
    
    
    
```Csharp

[TypeFilter(typeof(SimpleResourceFilter))]
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

```
    
Класс ServiceFilterAttribute в отличие от TypeFilterAttribute извлекает экземпляр фильтра напрямую из DI. Для этого при применении ему передается тип фильтра: 
    
```Csharp

[ServiceFilter(typeof(SimpleResourceFilter))]
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
    
```
    
Но чтобы данный фильтр стал доступен, его надо зарегистрировать в приложении в качестве сервиса:    
    
```Csharp
   
using MvcApp.Filters;
 
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
builder.Services.AddScoped<SimpleResourceFilter>();  // добавление фильтра в качестве зависимости
 
var app = builder.Build();
app.MapDefaultControllerRoute();
 
app.Run();
    
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
---





## Представление Views

### Представления Razor

**Замечание**: полноценной html-страницей представление все равно не является, потому что во время выполнения эти представления компилируются в сборки и уже затем используются для генерации html-страниц, которые видит пользователь в своем браузере.

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

Кроме обычных представлений и мастер-страниц в ASP.NET Core MVC также можно использовать частичные представления или partial views. Их отличительной особенностью является то, что их можно встраивать в другие обычные представления. Частичные представления могут использоваться также как и обычные, однако наиболее удобной областью их использования является рендеринг результатов AJAX-запроса. По своему действию частичные представления похожи на секции, которые использовались в прошлой теме, только их код выносится в отдельные файлы.

Частичные представления полезны для создания различных панелей веб-страницы, например, панели меню, блока входа на сайт, каких-то других блоков.

За рендеринг частичных представлений отвечает объект PartialViewResult, который возвращается методом PartialView()

Замечание: по своему действию частичное представление похоже на обычное, только для него по умолчанию не определяется мастер-страница.


```csharp
@await Html.PartialAsync("_GetMessage")
```

### Внедрение зависимостей

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

---
Примечание:  Статические переменные класса - это способ хранить состояние объекта в памяти (во время работы приложения)

### Управление привязкой модели
https://metanit.com/sharp/aspnetmvc/5.3.php













### Внедрение зависимостей в представления


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

 ---
 ### Работа с формами и html
 https://metanit.com/sharp/aspnetmvc/3.8.php
 ---
 
    
    
## Маршрутизация
    
- MapControllerRoute() определяет произвольный маршрут и принимает следующие параметры:

```Csharp
MapControllerRoute(string name, string pattern, [object defaults = null], [object constraints = null], [object dataTokens = null]) 
```

- MapDefaultControllerRoute() определяет стандартный маршрут, фактически эквивалентен вызову
    
```Csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```
Поскольку это довольно часто используемое определение маршрута, то для него и был введен отдельный метод.    
    
 - MapAreaControllerRoute() определяет маршрут, который также учитывает область приложения. Имеет следующие параметры:   

```Csharp
MapAreaControllerRoute(string name, string areaName, string pattern, [object defaults = null], [object constraints = null], [object dataTokens = null])
```
    
Обязательный параметр areaName позволяет определить область, с которой будет связан маршрут.  
    
 - MapControllers() сопоставляет действия контроллера с запросами, используя маршрутизацию на основе атрибутов.   

- MapFallbackToController() определяет действие контроллера, которое будет обрабатывать запрос, если все остальые определенные маршруты не соответствуют запросу. Принимает имя контроллера и его метода:
 
```Csharp
MapFallbackToController(string action, string controller)
```
    

Для установки значений по умолчанию также можно применять параметр defaults метода MapControllerRoute(). Этот параметр представляет объект, свойства которого соответствуют параметрам маршрута. Например, определим следующий маршрут:    
    
```Csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
 
var app = builder.Build();
 
app.MapControllerRoute(
    name: "default", 
    pattern: "{action}",
    defaults: new { controller = "Home", action = "Index"});
 
app.Run(); 
```

### Ограничения маршрутов
    
Для параметров маршрута в MVC, также как и в общем в ASP.NET Core, можно устанавливать ограничения.
Ограничения можно установить непосредственно в шаблоне маршрута:    
    
```Csharp
 app.MapControllerRoute(
    name: "default", 
    pattern: "{controller=Home}/{action=Index}/{id:int?}");
```
 В данном случае указывается, что параметр id может иметь либо значение типа int, либо значение null   
    
  Второй способ установки ограничений представляет параметр constraints метода MapControllerRoute:  
 
 ```Csharp
 using Microsoft.AspNetCore.Routing.Constraints; // для типа IntRouteConstraint
 
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
 
var app = builder.Build();
 
app.MapControllerRoute(
    name: "default", 
    pattern: "{controller=Home}/{action=Index}/{id?}",
    constraints: new {id= new IntRouteConstraint()});  // ограничения маршрутов
 
app.Run();
 ```

Параметр constraints принимает объект, в котором свойства соответствуют по названиям параметрам маршрутов, а значения свойств - ограничения, применяемые к одноименным параметрам маршрутов. Так, в данном случае к параметру id применяется ограничение IntRouteConstraint, которое указывает, что id должно представлять значение типа int.   
    
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
     
### Null условная проверка с объединением

```csharp
string name = р?.Name ?? "No name";
```
    
--- 
 
### Настройка Postman для localhost

Finally fixed it by turning off File > Settings > General > SSL Certificate Verification

--- 

### Миграции

```dotnet ef migrations add Itinial```

```dotnet ef database update```

---
    
## dotnet cli

    
Веб-приложение
```dotnet new webapp -o aspnetcoreapp```

MVC
```dotnet new mvc -o aspnetcoreapp```    
    
Все шаблоны
```dotnet new --list```
    
Установка доверия к сертификату разработки
```csharp
dotnet dev-certs https --trust
```    
    
```dotnet watch run```  
    
 ```dotnet run```
    
```cmd
dotnet new webapi -o TodoApi
cd TodoApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory
code -r ../TodoApi
```

## Компоненты представлений

View Component или компонент представлений представляет код, который объединяет логику на языке C# и связанную с ней разметку razor и который решает какую-то определенную задачу, например, создание динамических меню, облако тегов, панель входа на сайт, корзина покупок и так далее.

View Component состоит из двух частей: класса на C# и частичного представления, которое вызывает методы этого класса. При этом View Component не может обрабатывать HTTP-запросы, а генерируемый им контент включается в код родительского представления, в котором вызывается компонент.

View Component поддерживает внедрение зависимостей, поэтому в коде view component можно получить зависимости из провайдера сервисов. В то же время комопонент не является частью жизненного цикла контроллера, поэтому к нему нельзя применять фильтры.


### Создание компоненты 
    
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
 
### Вызов компонента в представлении
    
Теперь создадим представление, которое будет выводить переданные из компонента данные. Если при вызове метода View в компоненте имя представления явным образом не указано, то в качестве представления по умолчанию используется файл Default.cshtml. Для поиска файла представления Razor будет просматривать следующие пути в порядке приоритета:

- Views/Название_Контроллера/Components/Название_Компонента/Название_Представления.cshtml

- Views/Shared/Components/Название_Компонента/Название_Представления.cshtml
    
    
```Csharp
@{

    User user = new UserApp.Models.User() { Id = 1, Name = "1", Age = 3 };
}

@await Component.InvokeAsync("Timer", new { user = user })

```

### Второй способ

```Csharp
<vc:timer user = "user" />

```
 Однако перед использованием тега нобходимо зарегистрировать наш компонент в представлении качестве tag-хелпера с помощью директивы @addTagHelper:   
 
 ```Csharp
@addTagHelper *, [Название_сборки_проекта]
 ```
 
При использовании тег-хелпера параметры компонента определяются как атрибуты тега, которым присваивается необходимое значение. Причем если у нас исползуется camelcase, при котором каждое подслово в составе составного слова пишется с большой буквы, например, includeSeconds, то в названии атрибута все подслова разделяются дефисом и начинаются со строчной буквы.
    
    
---

## Сортировка

 ```Csharp
 
    
public enum SortState
{
    NameAsc,    // по имени по возрастанию
    NameDesc,   // по имени по убыванию
    AgeAsc, // по возрасту по возрастанию
    AgeDesc,    // по возрасту по убыванию
    CompanyAsc, // по компании по возрастанию
    CompanyDesc // по компании по убыванию
}
    
    
 public async Task<IActionResult> SortUsers(SortState sortOrder)
    {

        IQueryable<User>? users = db.Users.Include(x => x.Company);

        ViewData["NameSort"] = sortOrder == SortState.NameAsc ? SortState.NameDesc : SortState.NameAsc;
        ViewData["AgeSort"] = sortOrder == SortState.AgeAsc ? SortState.AgeDesc : SortState.AgeAsc;
        ViewData["CompSort"] = sortOrder == SortState.CompanyAsc ? SortState.CompanyDesc : SortState.CompanyAsc;


        users = sortOrder switch
        {
            SortState.NameDesc => users.OrderByDescending(s => s.Name),
            SortState.AgeAsc => users.OrderBy(s => s.Age),
            SortState.AgeDesc => users.OrderByDescending(s => s.Age),
            SortState.CompanyAsc => users.OrderBy(s => s.Company!.Name),
            SortState.CompanyDesc => users.OrderByDescending(s => s.Company!.Name),
            _ => users.OrderBy(s => s.Name),
        };



        return View(await users.AsNoTracking().ToListAsync());
    }
    
 ```

### Сортировка (view)

```html
    
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using Learning.Models
@model IEnumerable<User>

<h1>Список пользователей</h1>
<table>
    <tr>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewData["NameSort"]">
                Имя
            </a>
        </th>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewBag.AgeSort">
                Возраст
            </a>
        </th>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewBag.CompSort">
                Компания
            </a>
        </th>
        @foreach (User u in Model)
        {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
        }
</table>
    
```

### helper for sorting
 
```Csharp


using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.Routing;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Razor.TagHelpers;
using Learning.Models;

namespace Learning.TagHelpers
{
    public class SortHeaderTagHelper : TagHelper
    {
        public SortState Property { get; set; } // значение текущего свойства, для которого создается тег
        public SortState Current { get; set; }  // значение активного свойства, выбранного для сортировки
        public string? Action { get; set; }  // действие контроллера, на которое создается ссылка
        public bool Up { get; set; }    // сортировка по возрастанию или убыванию

        [ViewContext]
        [HtmlAttributeNotBound]
        public ViewContext ViewContext { get; set; } = null!;

        IUrlHelperFactory urlHelperFactory;
        public SortHeaderTagHelper(IUrlHelperFactory helperFactory)
        {
            urlHelperFactory = helperFactory;
        }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            IUrlHelper urlHelper = urlHelperFactory.GetUrlHelper(ViewContext);
            output.TagName = "a";
            string? url = urlHelper.Action(Action, new { sortOrder = Property });
            output.Attributes.SetAttribute("href", url);
            // если текущее свойство имеет значение CurrentSort
            if (Current == Property)
            {
                TagBuilder tag = new TagBuilder("i");
                tag.AddCssClass("glyphicon");

                if (Up == true)   // если сортировка по возрастанию
                    tag.AddCssClass("glyphicon-chevron-up");
                else   // если сортировка по убыванию
                    tag.AddCssClass("glyphicon-chevron-down");

                output.PreContent.AppendHtml(tag);
            }
        }
    }
}
    
```

## Фильтрация
 
### Viewmodel
 
```Csharp
    
    using Microsoft.AspNetCore.Mvc.Rendering;
using System.Collections.Generic;


namespace Learning.Models
{
    public class FilterViewModel
    {
        public FilterViewModel(List<Company> companies, int company, string name)
        {
            // устанавливаем начальный элемент, который позволит выбрать всех
            companies.Insert(0, new Company { Name = "Все", Id = 0 });
            Companies = new SelectList(companies, "Id", "Name", company);
            SelectedCompany = company;
            SelectedName = name;
        }
        public SelectList Companies { get; } // список компаний
        public int SelectedCompany { get; } // выбранная компания
        public string SelectedName { get; } // введенное имя
    }
}

    
```
    
    
```Csharp
    
    public async Task<IActionResult> FilterUsers(int company)
    {

        IQueryable<User> users = db.Users.Include(p => p.Company);

        if (company != null && company != 0)
        {
            users = users.Where(u => u.CompanyId == company);
        }



        List<Company> companies = db.Companies.ToList();
        // устанавливаем начальный элемент, который позволит выбрать всех
        companies.Insert(0, new Company { Name = "Все", Id = 0 });



        FilterUsersModel filterModel = new FilterUsersModel
        {
            Users = users.ToList(),
            Companies = new SelectList(companies, "Id", "Name", company),
            
        };


        return View(filterModel);
    }
```

### Фильтрация (view)
 
```htnl
   
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using Learning.Models

@model FilterUsersModel


<form method="get">
    <div>
        <label>Компания: </label>
        <select name="company" asp-items="Model.Companies"></select>

        <input type="submit" value="Фильтр" />
    </div>
</form>



<h1>Список пользователей</h1>
<table>
    <tr>
        <th>
            
                Имя
         
        </th>
        <th>
            
                Возраст
        
        </th>
        <th>
            
                Компания
         
        </th>
        @foreach (User u in Model.Users)
        {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
        }
</table>
    
```

## Поиск
 
### Viewmodel
 
```Csharp
using Microsoft.AspNetCore.Mvc.Rendering;

namespace Learning.Models
{
    public class SearchUsersModel
    {
        public IEnumerable<User> Users { get; set; } = new List<User>();
        public string Name { get; set; }
    }
}
```
    
    
```Csharp
 
    public async Task<IActionResult> SearchUsers(string name)
    {
        IQueryable<User> users = db.Users.Include(p => p.Company);

        if (!string.IsNullOrEmpty(name))
        {
            users = users.Where(p => p.Name!.Contains(name));
        }


        SearchUsersModel searchModel = new SearchUsersModel
        {
            Users = users.ToList(),
            Name = name
        };


        return View(searchModel);
    }
    
```

### Поиск (view)
 
```html
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
    @using Learning.Models

    @model SearchUsersModel


<form method="get">
    <div>
        <label>Имя: </label>

        <input asp-for="Name" />

       

        <input type="submit" value="Поиск" />
    </div>
</form>



<h1>Список пользователей</h1>
<table>
    <tr>
        <th>
            
                Имя
         
        </th>
        <th>
            
                Возраст
        
        </th>
        <th>
            
                Компания
         
        </th>
        @foreach (User u in Model.Users)
        {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
        }
</table>    
```

## Пагинация

### Viewmodel
    
```Csharp
  using Microsoft.AspNetCore.Mvc.Rendering;
using System.Collections.Generic;


namespace Learning.Models
{
    public class PageUsersModel
    {

        public IEnumerable<User> Users { get; set; }
        public int PageNumber { get; }
        public int TotalPages { get; }
        public bool HasPreviousPage => PageNumber > 1;
        public bool HasNextPage => PageNumber < TotalPages;

    

        public PageUsersModel(int count, int pageNumber, int pageSize)
        {
            PageNumber = pageNumber;
            TotalPages = (int)Math.Ceiling(count / (double)pageSize);
        }


        

    }
}

```
    
```Csharp
    public async Task<IActionResult> PageUsers(int page = 1)
    {

        int pageSize = 3;   // количество элементов на странице

        IQueryable<User> source = db.Users.Include(x => x.Company);

        var count = await source.CountAsync();
        var items = await source.Skip((page - 1) * pageSize).Take(pageSize).ToListAsync();




        PageUsersModel pageViewModel = new PageUsersModel(count, page, pageSize)
        {
            Users = items
        };



        return View(pageViewModel);
    }
```
    
### Пагинация (view)
  
```html
   @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, Learning

@using Learning.Models

@model PageUsersModel

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" />


<style>
    .glyphicon {
        display: inline-block;
        padding: 0 5px;
    }

    .glyphicon-chevron-right:after {
        content: "\00BB";
    }

    .glyphicon-chevron-left:before {
        content: "\00AB";
    }
</style>



<h1>Список пользователей</h1>

<table class="table">
    <tr><th>Имя</th><th>Возраст</th><th>Компания</th></tr>
    @foreach (User u in Model.Users)
    {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
    }
</table>

<p>
    @if (Model.HasPreviousPage)
    {
        <a asp-action="PageUsers" asp-route-page="@(Model.PageNumber - 1)" class="glyphicon glyphicon-chevron-left">
            Назад
        </a>
    }
    @if (Model.HasNextPage)
    {
        <a asp-action="PageUsers" asp-route-page="@(Model.PageNumber + 1)" class="glyphicon glyphicon-chevron-right">
            Вперед
        </a>
    }
</p>

 
```
  
### helper for sorting, filter, search and pagination
  
```Csharp
   using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.Routing;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Razor.TagHelpers;
using MvcApp.Models;
 
namespace MvcApp.TagHelpers
{
    public class PageLinkTagHelper : TagHelper
    {
        private IUrlHelperFactory urlHelperFactory;
        public PageLinkTagHelper(IUrlHelperFactory helperFactory)
        {
            urlHelperFactory = helperFactory;
        }
        [ViewContext]
        [HtmlAttributeNotBound]
        public ViewContext ViewContext { get; set; } = null!;
        public PageViewModel? PageModel { get; set; }
        public string PageAction { get; set; } = "";
 
        [HtmlAttributeName(DictionaryAttributePrefix = "page-url-")]
        public Dictionary<string, object> PageUrlValues { get; set; } = new();
 
        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            if(PageModel == null) throw new Exception("PageModel is not set");
            IUrlHelper urlHelper = urlHelperFactory.GetUrlHelper(ViewContext);
            output.TagName = "div";
 
            // набор ссылок будет представлять список ul
            TagBuilder tag = new TagBuilder("ul");
            tag.AddCssClass("pagination");
 
            // формируем три ссылки - на текущую, предыдущую и следующую
            TagBuilder currentItem = CreateTag(PageModel.PageNumber, urlHelper);
 
            // создаем ссылку на предыдущую страницу, если она есть
            if (PageModel.HasPreviousPage)
            {
                TagBuilder prevItem = CreateTag(PageModel.PageNumber - 1, urlHelper);
                tag.InnerHtml.AppendHtml(prevItem);
            }
 
            tag.InnerHtml.AppendHtml(currentItem);
            // создаем ссылку на следующую страницу, если она есть
            if (PageModel.HasNextPage)
            {
                TagBuilder nextItem = CreateTag(PageModel.PageNumber + 1, urlHelper);
                tag.InnerHtml.AppendHtml(nextItem);
            }
            output.Content.AppendHtml(tag);
        }
 
        TagBuilder CreateTag(int pageNumber, IUrlHelper urlHelper)
        {
            TagBuilder item = new TagBuilder("li");
            TagBuilder link = new TagBuilder("a");
            if (pageNumber == PageModel?.PageNumber)
            {
                item.AddCssClass("active");
            }
            else
            {
                PageUrlValues["page"] = pageNumber;
                link.Attributes["href"] = urlHelper.Action(PageAction, PageUrlValues);
            }
            item.AddCssClass("page-item");
            link.AddCssClass("page-link");
            link.InnerHtml.Append(pageNumber.ToString());
            item.InnerHtml.AppendHtml(link);
            return item;
        }
    }
} 
```
    
    
