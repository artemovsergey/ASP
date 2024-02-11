# ASP Core

## Паттерны проектирования

![](/Patterns.png)


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
builder.Services.AddTransient<IRepository,DataRepository>(); // одие раз создается одиночный объект для всего приложенияы
builder.Services.AddTransient<ICategoryRepository, CategoryRepository>();
builder.Services.AddTransient<IOrderRepository, OrderRepository>();

builder.Services.AddCors();



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
app.UseCors(x => x.AllowAnyHeader().AllowAnyMethod().WithOrigins("http://localhost:4200"));


//app.UseNodeModules();

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
  }
}

```

# Scaffold

В консоли диспетчера пакетов

```Scaffold-DbContext "Server=localhost;Database=Users;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models```

# dotnet cli

- dotnet new gitignore
- dotnet tool install --global dotnet-ef
- dotnet ef database drop --force
- dotnet ef migrations add InitialCreate
- dotnet ef database update
- dotnet dev-certs https --trust

# Переадресация

Протокол HTTP поддерживает два типа переадресации:

- постоянная переадресация. При постоянной переадресации сервер будет отправлять браузеру статусный код 301. При данном типе переадресации предполагается, что запрашиваемый документ окончательно перемещен в другое место. И после получения статусного кода 301 браузер может автоматически настраивать запросы на новый ресурс, даже если старый ресурс со временем перестанет применять переадресацию. Поэтому данный способ можно использовать, если вы полностью уверены, что документ на старое место уже не возвратится..

- временная переадресация. При временной переадресации сервер будет отправлять браузеру статусный код 302. При этом считается, что запрашиваемый документ временно перемещен на другую страницу.

В обоих случаях для создания переадресации может использоваться объект ```RedirectResult```, однако метод, возвращающий данный объект, будет отличаться.

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

     
# Null условная проверка с объединением

```csharp
string name = р?.Name ?? "No name";
```

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


# Фильтрация, сортировка

```Csharp
public async Task<IEnumerable<News>> GetNews(int pageIndex,
                                            int pageSize,
                                            string sortColumn,
                                            string sortOrder,
                                            string filterColumn,
                                            string filterQuery)
{

    if (!string.IsNullOrEmpty(sortColumn) && IsValidProperty(sortColumn))
    {
        sortOrder = !string.IsNullOrEmpty(sortOrder) && sortOrder.ToUpper() == "ASC"
        ? "ASC"
        : "DESC";
    }

    IQueryable<News> newsList = _db.NewsList;

    if (!string.IsNullOrEmpty(filterColumn)
        && !string.IsNullOrEmpty(filterQuery)
        && IsValidProperty(filterColumn))
    {
        //users = users.Where(u => $"{filterColumn}".Contains($"{filterQuery})"));
        newsList = newsList.Where($"{filterColumn}.Contains(@0)", filterQuery);

        Console.WriteLine($"Фильтрация: {newsList.Count()}");
    }

    return await newsList.OrderBy($"{sortColumn} {sortOrder}")
                          .Skip(pageIndex * pageSize)
                          .Take(pageSize)

                          .ToListAsync();
}

public static bool IsValidProperty(string propertyName,
                              bool throwExceptionIfNotFound = true)
{
    var prop = typeof(News).GetProperty(
    propertyName,

    BindingFlags.IgnoreCase |
    BindingFlags.Public |
    BindingFlags.Instance);
    if (prop == null && throwExceptionIfNotFound)
        throw new NotSupportedException(
        string.Format($"ERROR: Property '{propertyName}' does not exist.")
     );
    return prop != null;

}

```

# Настройка формата даты для PostgreSQL в файле Context.cs

```
    static NewsAggregationContext()
    {
        AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);
    }
```

# Hash md5

```Csharp
    public async Task<string> HashNews(string title)
    {
        // Реализация хеширования пароля с использованием MD5
        using (MD5 md5 = MD5.Create())
        {
            byte[] inputBytes = Encoding.ASCII.GetBytes(title);
            byte[] hashBytes = md5.ComputeHash(inputBytes);

            // Конвертируем байты обратно в строку
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < hashBytes.Length; i++)
            {
                sb.Append(hashBytes[i].ToString("X2"));
            }
            return sb.ToString();
        }
    }
```

# Parsing XML

```Csharp
    private SyndicationFeed Fetch(string url)
    {

        using (var client = new WebClient())
        {
            var rssData = client.DownloadString(url);
            using (var reader = XmlReader.Create(new StringReader(rssData)))
            {
                var feed = SyndicationFeed.Load(reader);
                return feed;
            }
        }
    }
```

# API Result для фильтрации, сортировки и пагинации

```Csharp
public class ApiResult<T>
{

    public List<T> Data { get; private set; }
    public int PageIndex { get; private set; }
    public int PageSize { get; private set; }
    public int TotalCount { get; private set; }
    public int TotalPages { get; private set; }
    public bool HasPreviousPage
    {
        get
        {
            return (PageIndex > 0);
        }
    }
    public bool HasNextPage
    {
        get
        {
            return ((PageIndex + 1) < TotalPages);
        }
    }
    public string? SortColumn { get; set; }
    public string? SortOrder { get; set; }
    public string FilterColumn { get; set; }
    public string FilterQuery { get; set; }

    public ApiResult(List<T> data,
                     int count,
                     int pageIndex,
                     int pageSize,
                     string? sortColumn,
                     string? sortOrder,
                     string filterColumn,
                     string filterQuery)
    {
        Data = data;
        PageIndex = pageIndex;
        PageSize = pageSize;
        TotalCount = count;
        TotalPages = (int)Math.Ceiling(count / (double)pageSize);
        SortColumn = sortColumn;
        SortOrder = sortOrder;
        FilterColumn = filterColumn;
        FilterQuery = filterQuery;
    }
}
```

# Валидация

```Csharp
    public class UserValidator : AbstractValidator<User>
    {
        public UserValidator()
        {
            RuleFor(user => user.Email)
                .NotEmpty().WithMessage("Email is required.")
                .EmailAddress().WithMessage("A valid email address is required.");

            // Добавьте другие правила валидации здесь...
        }
    }  
```

Program.cs
```Csharp
builder.Services.AddScoped<IValidator<User>, UserValidator>();
```

# Проверка валидатора

```Csharp
        private readonly IValidator<User> _userValidator;
        var validationResult = _userValidator.Validate(user);

        if (!validationResult.IsValid)
        {
            foreach (var failure in validationResult.Errors)
            {
                ModelState.AddModelError(failure.PropertyName, failure.ErrorMessage);
            }

            return BadRequest(ModelState);
        }
```



# Настройка отображения json для API
Program.cs
```Csharp
builder.Services.AddControllers()
 .AddJsonOptions(options =>
 {
     // влияет на производительность
     options.JsonSerializerOptions.WriteIndented = true;
 });
```

# Запрос на проверку уникальности

```Csharp
    [HttpPost]
    [Route("isUniqName")]
    public async Task<bool> IsNameUniq(string role)
    {
       return await _db.Roles.AnyAsync(r => r.Name == role);
    }
```

# Тестовые данные

```Csharp
 [HttpGet("/generate")]
 public async Task<IActionResult> SeedUsers()
 {

     var faker = new Faker<User>()
     //.RuleFor(u => u.Id, f => f.UniqueIndex)
     .RuleFor(u => u.Email, f => f.Internet.Email())
     .RuleFor(u => u.Login, f => f.Person.UserName)
     .RuleFor(u => u.Password, f => f.Internet.Password());

     List<User> users = faker.Generate(100);

     using (var context = new KeeperContext())
     {
         context.Users.AddRangeAsync(users);
         await context.SaveChangesAsync();
     }

     return Ok();
 }
```

