# Фильтры
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
 
**Замечание**: При реализации интерфейсов следует учитывать, что мы можем реализовать либо только синхронный, либо только асинхронный вариант. Если же класс будет реализовать оба варианта, то система будет вызывать только метод асинхронного интерфейса, а реализация синхронного интерфейса будет игнорироваться.
   
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
    
**Замечание**: Также стоит отметить, что класс фильтра также наследует класс Attribute, так как фильтры реализуются в основном именно как атрибуты, что упрощает их применение.  

**Применение**: Таким образом, фильтры позволяют нам избежать повторения в соответствии принципом DRY (Don't Repeat Yourself)
    
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

    
 # Область действия фильтров
    
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


# Переопределение порядка фильтров
 
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
    

# Передача параметров в фильтры
    
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
 
# Фильтры и Dependency Injection
    

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
