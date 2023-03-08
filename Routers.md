# Маршрутизация
    
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

# Ограничения маршрутов
    
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
    
# Маршрутизация на основе аттрибутов

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

# Атрибуты маршрутизации в ASP.NET MVC Core
 
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
 
# Использование префиксов

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
# Параметры controller и action
 
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
