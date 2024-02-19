# Razor Pages

Program.cs
```Csharp
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        builder.Services.AddRazorPages();
        builder.Services.AddHealthChecks();
        #if DEBUG
        builder.Services.AddSassCompiler();
        #endif

        var app = builder.Build();

        //app.UseWelcomePage();

        #region Middleware Error
        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseExceptionHandler("/Error");
            app.UseHsts();
        } 
        #endregion

        app.UseHttpsRedirection();

        app.UseStaticFiles();

        app.UseRouting();
        app.UseAuthorization();

        /*  Регистрируем все
            страницы Razor
            в приложении
            в качестве
            конечных точек. */

        app.MapRazorPages();

        /* Регистрируем встроенную конечную точку, которая
           возвращает «Hello World!» на маршруте /test. */

        app.MapGet("/test", async context =>
        {
            await context.Response.WriteAsync("Hello World!");
        });


        /* Регистрируем конечную точку проверки
           работоспособности на маршруте /healthz. */
        app.MapHealthChecks("/healthz");




        app.Run();
    }
}
```

# Scss

Подключение кастомного custom.scss. Тут надо подключиться пакет ```AspCore.Sass.Compiler```

Program.cs:
```Csharp
#if DEBUG
builder.Services.AddSassCompiler();
#endif
```
appsettings.json:

```json
  "SassCompiler": {
    "SourceFolder": "wwwroot/scss",
    "TargetFolder": "wwwroot/css",
    "Arguments": "--style=compressed",
    "GenerateScopedCss": true,
    "ScopedCssFolders": [ "Views", "Pages", "Shared", "Components" ],
    "IncludePaths": []
  }
```

# Handler

```Csharp
        // On{verb}[handler][Async]
        public void OnGet()
        {
            var url = Url.Page("./Sign");
            Console.WriteLine(url);
        }

        /* https://localhost:7246/?handler=SayHello как параметр строки запроса */
        /* https://localhost:7246/SayHello  как параметр маршрута @page "{handler}" */

        public void OnGetSayHello()
        {
            Console.WriteLine("Hello");
        }
```

# Routing

Page.cshtml
```html
@page "{category}/{name:length(3)}" 
<!-- добавление дополнительных сегментов с параметрами маршрута -->
<!-- / заменяет маршрут по умолчанию  -->
<!-- универсальные параметры *other или **other-->
```

# Binding Model

Binding to Property
```Csharp

        [BindProperty(SupportsGet = true)]
        public string user { get; set; } = "User";


        public void OnGet()
        {
            ViewData["user"] = $"Пользователь из запроса: {this.user}";            
        }
```

Три источника привязки по умолчанию, чтобы найти соответствие вашим моделям привязки:
1. значения формы;
2. значения маршрута;
3. значения строки запроса.

Привязка загрузки файлов с помощью IFormFile
Привязка коллекций и словарей

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


