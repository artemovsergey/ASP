# ASP.Net Core

## Tехнологии

### ASP.Net Core 6
### EntityFrameworkCore (6.0.7)

---

### Быстрое создание прототипа MVC

```dotnet new mvc --output TestMVC```


### Установить CLI-инструмент Scaffolding

```dotnet tool install -g dotnet-aspnet-codegenerator```

```dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design```

```dotnet add package Microsoft.EntityFrameworkCore.Design```

```dotnet tool install --global dotnet-ef```

```dotnet add package Microsoft.EntityFrameworkCore.SQLite```

```dotnet add package Microsoft.EntityFrameworkCore.SqlServer```

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

### Добавить службу контекста

```Csharp
builder.Services.AddDbContext<BeerContext>();
```
### Сделать скаффолдинг

```Csharp
dotnet aspnet-codegenerator controller --controllerName Home --model Beer --dataContext BeerContext --useDefaultLayout -outDir Controllers -f --useSqlite
```

### Выполнить миграции в базу данных

```dotnet ef migrations add InitialCreate```

**Замечание**: --project Name, если несколько проекто. Помогает также очистка и пересборка решения.

```dotnet ef database update```

**Замечание**: --connection "Data Source=My.db"

```dotnet run```


**Замечание**: это можно только для тренировки.

---

### Теория

### MVC
Цель паттерна MVC - разделение приложения на три функциональных области, каждая из которых может содержать и логиу и данные. Цель не в том, чтобы устранить логику из модели. Наоборот, цель в том, чтобы гарантировать наличие в модели только логики предназначенной для создания и управления данными

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
using TestASP.Models;
using Microsoft.Extensions.FileProviders;

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
string connection = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<TestStoreContext>(options => options.UseSqlServer(connection));


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

// тоже самое
//app.MapDefaultControllerRoute();

IHostEnvironment? env = app.Services.GetService<IHostEnvironment>();
if (env != null)
{
    // добавляем поддержку каталога node_modules
    app.UseFileServer(new FileServerOptions()
    {
        FileProvider = new PhysicalFileProvider(
            Path.Combine(env.ContentRootPath, "node_modules")
        ),
        RequestPath = "/node_modules",
        EnableDirectoryBrowsing = false
    });
}

/*
 Последний вызов app.UseFileServer() позволит сопоставлять 
все запросы с "/node_modules" с каталогом "node_modules".

В реальности, конечно, в большинстве случаев для библиотек из node_modules 
используют минификацию/бандлинг с помощью BundleConfig из первой 
темы этой главы с последующим копированием в папку wwwroot, 
поэтому не придется прибегать к проекции запросов на каталог node_modules. 
Но тем не менее так мы тоже можем обращаться к файлам в каталоге node_modules.

 */






app.Run();
//

public class AppTimeService { };


```


## Асинхронный CRUD

### Index

```Csharp
     private readonly TestStoreContext _context;

        public UsersController(TestStoreContext context)
        {
            _context = context;
        }

        // GET: Users
        public async Task<IActionResult> Index()
        {
              return _context.Users != null ? 
                          View(await _context.Users.ToListAsync()) :
                          Problem("Entity set 'TestStoreContext.Users'  is null.");
        }
```

### Details id

```Csharp
  // GET: Users/Details/5
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null || _context.Users == null)
            {
                return NotFound();
            }

            var user = await _context.Users
                .FirstOrDefaultAsync(m => m.Id == id);
            if (user == null)
            {
                return NotFound();
            }

            return View(user);
        }
```

### Create

```Csharp
        // POST: Users/Create
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("Id,Name,Age")] User user)
        {
            if (ModelState.IsValid)
            {
                _context.Add(user);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(user);
        }
```
Замечание: полезный код

```Csharp
    string errors = $"Количество ошибок: {ModelState.ErrorCount}. Ошибки в свойствах: ";
    foreach(var prop in ModelState.Keys)
    {
        errors = $"{errors}{prop}; ";
    }
```


### Edit

```Csharp
        // GET: Users/Edit/5
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null || _context.Users == null)
            {
                return NotFound();
            }

            var user = await _context.Users.FindAsync(id);
            if (user == null)
            {
                return NotFound();
            }
            return View(user);
        }
```

### Edit POST

```Csharp
        // POST: Users/Edit/5
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int? id, [Bind("Id,Name,Age")] User user)
        {
            if (id != user.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(user);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!UserExists(user.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Index));
            }
            return View(user);
        }
        
        private bool UserExists(int? id)
        {
          return (_context.Users?.Any(e => e.Id == id)).GetValueOrDefault();
        }
        
```

### Delete

```Csharp
        // GET: Users/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null || _context.Users == null)
            {
                return NotFound();
            }

            var user = await _context.Users
                .FirstOrDefaultAsync(m => m.Id == id);
            if (user == null)
            {
                return NotFound();
            }

            return View(user);
        }
```

### Delete POST

```Csharp
        // POST: Users/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int? id)
        {
            if (_context.Users == null)
            {
                return Problem("Entity set 'TestStoreContext.Users'  is null.");
            }
            var user = await _context.Users.FindAsync(id);
            if (user != null)
            {
                _context.Users.Remove(user);
            }
            
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
```

### Index.cshtml

```html
@model IEnumerable<TestASP.Models.User>

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>
<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Name)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Age)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Name)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Age)
            </td>
            <td>
                <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                <a asp-action="Details" asp-route-id="@item.Id">Details</a> |
                <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>

```

### Edit.cshtml

```html
@model TestASP.Models.User

@{
    ViewData["Title"] = "Edit";
}

<h1>Edit</h1>

<h4>User</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Edit">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="Id" />
            <div class="form-group">
                <label asp-for="Name" class="control-label"></label>
                <input asp-for="Name" class="form-control" />
                <span asp-validation-for="Name" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Age" class="control-label"></label>
                <input asp-for="Age" class="form-control" />
                <span asp-validation-for="Age" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Save" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

```

### Details.cshtml

```html
@model TestASP.Models.User

@{
    ViewData["Title"] = "Details";
}

<h1>Details</h1>

<div>
    <h4>User</h4>
    <hr />
    <dl class="row">
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Name)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Name)
        </dd>
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Age)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Age)
        </dd>
    </dl>
</div>
<div>
    <a asp-action="Edit" asp-route-id="@Model?.Id">Edit</a> |
    <a asp-action="Index">Back to List</a>
</div>

```

### Delete.cshtml

```html
@model TestASP.Models.User

@{
    ViewData["Title"] = "Delete";
}

<h1>Delete</h1>

<h3>Are you sure you want to delete this?</h3>
<div>
    <h4>User</h4>
    <hr />
    <dl class="row">
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Name)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Name)
        </dd>
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Age)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Age)
        </dd>
    </dl>
    
    <form asp-action="Delete">
        <input type="hidden" asp-for="Id" />
        <input type="submit" value="Delete" class="btn btn-danger" /> |
        <a asp-action="Index">Back to List</a>
    </form>
</div>

```

### Create.cshtml

```html
@model TestASP.Models.User

@{
    ViewData["Title"] = "Create";
}

<h1>Create</h1>

<h4>User</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Create">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Name" class="control-label"></label>
                <input asp-for="Name" class="form-control" />
                <span asp-validation-for="Name" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Age" class="control-label"></label>
                <input asp-for="Age" class="form-control" />
                <span asp-validation-for="Age" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Create" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

```



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
    
    
# Настройка webpack

Затем создайте новый каталог ```ClientApp``` в корне вашего проекта MVC. Внутри этого нового каталога создайте файл с именем ```package.json```.

    
## package.json
    
```json
    
{
  "name": "myapp-client-bundle",
  "version": "1.0.0",
  "description": "This is client-side scripts bundle for MyApp",
  "private": true,

  "scripts": {
    "build": "webpack --mode=development",
    "build:prod": "webpack --mode=production",
    "watch": "webpack --watch"
  },

  "devDependencies": {

    "ts-loader": "^9.2.5",
    "typescript": "^4.4.3",
    "webpack": "^5.52.1",
    "webpack-cli": "^4.8.0",
    "css-loader": "^6.7.1",

    "style-loader": "^3.3.1",
    "mini-css-extract-plugin": "^2.6.0"

  },
  "dependencies": {

    "@popperjs/core": "^2.11.2",
    "jquery": "^3.6.0",
    "jquery-validation": "^1.19.3",
    "jquery-validation-unobtrusive": "^3.2.12",
    "bootstrap": "5.2.0"
  }
}
    
```

Выполнить установку пакетов npm
```npm install```
    
 Появится новый ```node_modules``` каталог, содержащий огромное количество файлов JS и CSS. Считайте, что этот каталог похож на каталоги binи obj (или, что еще лучше, на кеш пакетов NuGet). Хотя он не содержит двоичных файлов, его содержимое по-прежнему представляет собой набор зависимостей, на которые мы ссылаемся из нашего собственного кода.
    
Создаем каталог ```ClinetApp``` в нем создаем папку ```src```, в ней создаем файлы, например ```index.js```.
    
```js
    
// JS Dependencies: Popper, Bootstrap & JQuery
import '@popperjs/core';
import 'bootstrap';
import 'jquery';
// Using the next two lines is like including partial view _ValidationScriptsPartial.cshtml
import 'jquery-validation';
import 'jquery-validation-unobtrusive';

// CSS Dependencies: Bootstrap
import 'bootstrap/dist/css/bootstrap.css';

// Custom JS imports
// ... none at the moment

// Custom CSS imports
import '../css/site.css';

console.log('The \'site\' bundle has been loaded!');
    
```

Строки со 2 по 9 будут импортировать код изнутри node_modules. Мы импортируем Javascript, Popper и CSS Bootstrap, а также несколько библиотек JQuery, которые используются сценариями проверки ASP.NET Core.
Все это делается с помощью модулей ECMAScript 6 — современного подхода к импорту JS-файлов из других JS-файлов или даже CSS-файлов из JS-файлов.

 
## webpack.config.js

 Создайте новый файл с именем ```webpack.config.js``` и поместите его в ClientAppкаталог.
    
```js

const path = require('path');
MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
    entry: {
        site: './src/js/site.js',
        bootstrap_js: './src/js/bootstrap_js.js',
        validation: './src/js/validation.js',
        index: './src/js/index.js',
        indexTs: './src/ts/index.ts'
    },

    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
    },

    output: {

        library: {
            name: 'MYAPP',
            type: 'var'
        },

        filename: '[name].entry.js',
        path: path.resolve(__dirname, '..', 'wwwroot', 'dist')
    },
    devtool: 'source-map',
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.css$/,
                //use: ['style-loader', 'css-loader'],
                use: [{ loader: MiniCssExtractPlugin.loader }, 'css-loader']
            },
            {
                test: /\.(eot|woff(2)?|ttf|otf|svg)$/i,
                type: 'asset'
            },

            {
                test: /\.tsx?$/,
                use: 'ts-loader',
                exclude: /node_modules/,
            }

        ]
    },

     plugins: [
     new MiniCssExtractPlugin({ filename: "[name].css"})
     ]

};
    
```

## tsconfig.json
 
```json
 
{
  "compilerOptions": {
    "outDir": "./dist/",
    "noImplicitAny": true,
    "module": "es6",
    "target": "es5",
    "allowJs": true,
    "moduleResolution": "node"
  }
}
    
```
    
## index.ts
  
```ts
export * from './hello';  
```
    
## hello.ts
  
```ts
export namespace funcs {
    export function hello(): void {
        const message = 'Hello world!';
        console.log(message);
    }
}
```    
 
Можно подключать
    
```js
<script src="/dist/indexTs.entry.js"></script>


<script>
MYAPP.funcs.hello();
</script>
```
    
    
Далее ```npm run build```
   

Подключить файл в макете:

```html
<script src="~/dist/site.entry.js" defer></script>
```

Скрипт для валидации asp форм:
 
```html
<script src="~/dist/validation.entry.js" defer></script>   
```
    
## Скрипт выпадающий список
  
```html
<div class="dropdown">
    <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
        Dropdown button
    </button>
    <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
        <li><a class="dropdown-item" href="#">Action</a></li>
        <li><a class="dropdown-item" href="#">Another action</a></li>
        <li><a class="dropdown-item" href="#">Something else here</a></li>
    </ul>
</div>
```
 
```
 <script src="~/dist/bootstrap_js.entry.js" defer></script>
```
 
Замечание: можно применить секции
  
```html
 @await RenderSectionAsync("Scripts", required: false)
```
 
```Csharp
    
    @section Scripts
    {
     <script src="~/dist/bootstrap_js.entry.js" defer></script>
    }
    
```

Для наблюдения за изменениями:
    
```npm run watch```
    
Для сборки webpack при сборке dotnet можно настроить csproject
    

```Csharp
<Project Sdk="Microsoft.NET.Sdk.Web">

     <PropertyGroup>
         <TargetFramework>net6.0</TargetFramework>
         <Nullable>disable</Nullable>
         <ImplicitUsings>enable</ImplicitUsings>
        <IsPackable>false</IsPackable>
        <MpaRoot>ClientApp\</MpaRoot>
       <WWWRoot>wwwroot\</WWWRoot>
        <DefaultItemExcludes>$(DefaultItemExcludes);$(MpaRoot)node_modules\**</DefaultItemExcludes>
     </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.SpaServices.Extensions" Version="6.0.3"/>
    </ItemGroup>

    <ItemGroup>
        <!-- Don't publish the MPA source files, but do show them in the project files list -->
        <Content Remove="$(MpaRoot)**"/>
        <None Remove="$(MpaRoot)**"/>
        <None Include="$(MpaRoot)**" Exclude="$(MpaRoot)node_modules\**"/>
    </ItemGroup>

    <Target Name="NpmInstall" BeforeTargets="Build" Condition=" '$(Configuration)' == 'Debug' And !Exists('$(MpaRoot)node_modules') ">
        <!-- Ensure Node.js is installed -->
        <Exec Command="node --version" ContinueOnError="true">
            <Output TaskParameter="ExitCode" PropertyName="ErrorCode"/>
        </Exec>
        <Error Condition="'$(ErrorCode)' != '0'" Text="Node.js is required to build and run this project. To continue, please install Node.js from https://nodejs.org/, and then restart your command prompt or IDE."/>
        <Message Importance="high" Text="Restoring dependencies using 'npm'. This may take several minutes..."/>
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm install"/>
    </Target>

    <Target Name="NpmRunBuild" BeforeTargets="Build" DependsOnTargets="NpmInstall">
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm run build"/>
    </Target>

    <Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish">
        <!-- As part of publishing, ensure the JS resources are freshly built in production mode -->
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm install"/>
       <Exec WorkingDirectory="$(MpaRoot)" Command="npm run build"/>

        <!-- Include the newly-built files in the publish output -->
       <ItemGroup>
            <DistFiles Include="$(WWWRoot)dist\**"/>
            <ResolvedFileToPublish Include="@(DistFiles->'%(FullPath)')" Exclude="@(ResolvedFileToPublish)">
                <RelativePath>%(DistFiles.Identity)</RelativePath>
                <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
                <ExcludeFromSingleFile>true</ExcludeFromSingleFile>
           </ResolvedFileToPublish>
        </ItemGroup>
    </Target>

    <Target Name="NpmClean" BeforeTargets="Clean">
        <RemoveDir Directories="$(WWWRoot)dist"/>
        <RemoveDir Directories="$(MpaRoot)node_modules"/>
   </Target>

 </Project>
```
    
    
    
В результате:
- Мы ссылаемся на наши библиотеки с определенным номером версии
- Зависимости не размещаются внутри дерева проекта
- В итоге мы получим прирост производительности (в стандартном шаблоне MVC на все Bootstrap и JQuery ссылаются со всех страниц).
Наша система сборки расширяема: Хотите Sass? Без проблем! Хотите использовать самые последние функции ECMAScript? Ты понял! Нужна минификация или обфускация? Без проблем!

















