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
