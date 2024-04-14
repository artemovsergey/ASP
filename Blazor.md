# Blazor

- dotnet new blazor -o ClientBlazorApp -e --interactivity WebAssembly
- dotnet new blazor -o ServerBlazorApp -e --interactivity None


## Подключение сервисов для интерактивного рендеринга на стороне клиента и сервера
```Csharp
builder.Services.AddRazorComponents()
    .AddInteractiveWebAssemblyComponents()
    .AddInteractiveServerComponents();
```

## Применение сервисов
```Csharp
app.MapRazorComponents<App>()
    .AddInteractiveWebAssemblyRenderMode()
    .AddInteractiveServerRenderMode()
    .AddAdditionalAssemblies(typeof(ClientBlazorApp.Client._Imports).Assembly);
```

## Router
```razor
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(Layout.MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
</Router>
```

## MainLayout

```razor
@inherits LayoutComponentBase
 
<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>
 
    <main>
        <div class="top-row px-4">
            <a href="https://learn.microsoft.com/aspnet/core/" target="_blank">About</a>
        </div>
 
        <article class="content px-4">
            @Body
        </article>
    </main>
</div>
 
<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

## _Imports.razor
```razor
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using static Microsoft.AspNetCore.Components.Web.RenderMode
```

## Компонент NavMenu
```razor
<div class="nav-scrollable" >
    <nav class="flex-column">
        <div class="nav-item px-3">
            <NavLink class="nav-link" href="" Match="NavLinkMatch.All">
                <span class="bi bi-house-door-fill-nav-menu" aria-hidden="true"></span> Home
            </NavLink>
        </div>

        <div class="nav-item px-3">
            <NavLink class="nav-link" href="sign">
                <span class="bi bi-plus-square-fill-nav-menu" aria-hidden="true"></span> Регистрация
            </NavLink>
        </div>

        <div class="nav-item px-3">
            <NavLink class="nav-link" href="weather">
                <span class="bi bi-list-nested-nav-menu" aria-hidden="true"></span> Госпитализация
            </NavLink>
        </div>
    </nav>
</div>
```

## Компонент App
```Csharp
<!DOCTYPE html>
<html>
    <head>
        <title></title>
        <base href="/" />
        <HeadOutlet />
    </head>
    <body>
 
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <RouteView RouteData="@routeData"  />
        </Found>
    </Router>
 
    <script src="_framework/blazor.web.js"></script>
    </body>
</html>
```

## Применение автоматического рендеринга в компоненте
```
@rendermode InteractiveAuto
```

## Partial class for component

Можно попробовать применить Code behind с помощью partial для компонента Blazor
То есть, если есть компонент User.razor, то можно добавить к нему User.razor.cs с partial

Для применения своего Layout в компоненте @layout name

@on{Dom Event} = "Delegate"

Значение по умолчанию для обработчиков событий 

```Csharp
<button @onclick = "Show">
@code{
 
  private void Show(MouseEventArgs e)
   {

   }
}
```

## Lambda
```Csharp
<button @onclick = "@(e => Show(e, buttonNumber)">
@code{
 
  private void Show(MouseEventArgs e, int ButtonNumber)
   {

   }
}
```

## Жизненынй цикл компонента

OnInitialized
OnParameterSet
OnAfterRender


## Передача события из дочернего компонента в родительский

Дочерний компонент:
```Csharp
<button @onclick = "NameEvent">
@code{
 
    [Parameter]
    public EventCallBack<MouseEventArgs> NameEvent {get; set;}
}
```

Родительский компонент:
```Csharp
<ChildComponent NameEvent = "NameHandler"></ChildComponent>
@code{
 
  private void NameHandler()
   {

   }
}
```

## Динамические компоненты
RenderFragment @ChildContent
DynamicComponent

## Обработка ошибок

<ErrorBoundary>
	<ChildContent>
 	
	</ChildContent>

        <ErrorContent>

        </ErrorContent>
</ErrorBoundary>


## Настройка клиента Http в API 

builder.Services.AddHttpClient<IServiceName, ServiceName>(c => c.BaseAddress = new Uri(builder.HostEnviroment.BaseAddress))

## Инкапсуляция вызовов API

Так можно отдельно вынести получение и отправку данных данных через API в методы

```Csharp
public async Task<IEnumerable<Employee>> GetAllEmployees(){

return await JsonSerializer.DeserializeAsync<IEnumerable<Employee>> (await _httpClien.GetStreamAsync(), new JsonSerializerOption() { PropertyNameCaseInsensitive = true } );
}
```

- SetItem()
- GetItem()
- ContainKey()
- RemoveItem()


## LocalStorage
using Blazored LocalStorage
Для хранения состояния в LocalStorege можно применить пакет
Blazored LocalStorege

```Csharp
@inject Blazored.LocalStorage.ILocalStorageService localStorage
var firstName = await localStorage.GetItemAsync<string>("EmployeeFirstName");
```

## Result
Есть библиотека для возврата Result

## Поиск
Результат обработки события поиска
```
<input @bind-value="Employee.LastName" @bind-value:event="oninput"/>
```

## Загрузка изображений
- рассмотреть файл про загрузку изображений

## Валидация
- Валидация
DataAnnotation
DataAnnotationValidator
ValidationSummary

Замечание: при работе с формами в компоненте надо поинмать с каким режимом мы работаетм. Если не указывать rendermode, то это обычные запросы, а не интерактивный режим

```Csharp
@page "/"
@page "/Sign"
@using SampleApp.Domen.Models
@using System.ComponentModel.DataAnnotations
@using SampleApp.Domen.Validations
<PageTitle> Регистрация </PageTitle>
@rendermode RenderMode.InteractiveServer

<div class="container text-center">
    <h3> Регистрация </h3>

    <div class="row border border-0 border-warning">
        <div class="col-6 offset-3 border border-0 border-primary">

            <EditForm method="post" EditContext="model" OnValidSubmit="@Submit" OnInvalidSubmit="@Submit">

                <DataAnnotationsValidator/>
                <ValidationSummary/>

                <InputText class="input-group" placeholder="Login" @bind-Value="p.Name" />
                <ValidationMessage For="@(() => p.Name)"/>

                <button type="submit">Submit</button>

            </EditForm>

        </div>
    </div>

</div>

@code {

    public Person p { get; set; } = new();
    EditContext model;

    protected override async Task OnInitializedAsync()
    {
        model = new(p);
    }

    public class Person()
    {
        [PersonNameValidator(new[] { "admin" })]
        [Required(ErrorMessage = "Имя должно быть не пустое")]
        public string Name { get; set; }
    }

    public async Task Submit()
    {
        Console.WriteLine($"Модель: {model.Validate()}");
    }
}

```

Пользовательский валидатор

```Csharp
using System.ComponentModel.DataAnnotations;

namespace SampleApp.Domen.Validations;

public class PersonNameValidator : ValidationAttribute
{
    string[] names;
    public PersonNameValidator(string[] names)
    {
        this.names = names;
    }
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (names.Contains(value?.ToString()))
        {
            return new ValidationResult("Некорректное имя!");
        }
        return ValidationResult.Success;
    }
}

```




## Вызор Js из Blazor
OnAfterRenderAsync


## Создание Razor Class Library
Также можно сделать Lazy Loading

# Тестирование компонентов

- bUnit

# ModelState

В методах контроллера API можно применять ModelState


# Загрузка изображения

Компонент
```Csharp
 private IBrowserFile selectedFile;
 protected async Task HandleValidSubmit()
 {
     Saved = false;

     if (Employee.EmployeeId == 0) //new
     {
         //image adding
         if (selectedFile != null)//take first image
         {
             var file = selectedFile;
             Stream stream = file.OpenReadStream();
             MemoryStream ms = new();
             await stream.CopyToAsync(ms);
             stream.Close();

             Employee.ImageName = file.Name;
             Employee.ImageContent = ms.ToArray();
         }

         var addedEmployee = await EmployeeDataService.AddEmployee(Employee);
         if (addedEmployee != null)
         {
             StatusClass = "alert-success";
             Message = "New employee added successfully.";
             Saved = true;
         }
         else
         {
             StatusClass = "alert-danger";
             Message = "Something went wrong adding the new employee. Please try again.";
             Saved = false;
         }
     }
     else
     {
         await EmployeeDataService.UpdateEmployee(Employee);
         StatusClass = "alert-success";
         Message = "Employee updated successfully.";
         Saved = true;
     }
 }
```

Метод API
```Csharp
 [HttpPost]
 public IActionResult CreateEmployee([FromBody] Employee employee)
 {
     if (employee == null)
         return BadRequest();

     if (employee.FirstName == string.Empty || employee.LastName == string.Empty)
     {
         ModelState.AddModelError("Name/FirstName", "The name or first name shouldn't be empty");
     }

     if (!ModelState.IsValid)
         return BadRequest(ModelState);

     //handle image upload
     //string currentUrl = _httpContextAccessor.HttpContext.Request.Host.Value;
     //var path = $"{_webHostEnvironment.WebRootPath}\\uploads\\{employee.ImageName}";
     //var fileStream = System.IO.File.Create(path);
     //fileStream.Write(employee.ImageContent, 0, employee.ImageContent.Length);
     //fileStream.Close();

    // employee.ImageName = $"https://{currentUrl}/uploads/{employee.ImageName}";

     var createdEmployee = _employeeRepository.AddEmployee(employee);

     return Created("employee", createdEmployee);
 }
```

# Модальное окно

```html
@if (_employee != null)
{
    <div class="modal fade show d-block" id="exampleModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="titleLabel">Employee Quick Add</h5>
                    <button type="button" class="close btn btn-lg" @onclick="@Close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <div class="col row">
                        <div class="col-12 col-sm-8">
                            <div class="form-group row">
                                <label class="col-sm-5 col-form-label">Employee ID</label>
                                <div class="col-sm-7">
                                    <label type="text" class="form-control-plaintext">@_employee.EmployeeId</label>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label class="col-sm-5 col-form-label">First name</label>
                                <div class="col-sm-7">
                                    <label type="text" readonly class="form-control-plaintext">@_employee.FirstName</label>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label class="col-sm-5 col-form-label">Last name</label>
                                <div class="col-sm-7">
                                    <label type="text" readonly class="form-control-plaintext">@_employee.LastName</label>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label class="col-sm-5 col-form-label">Birthdate</label>
                                <div class="col-sm-7">
                                    <label type="text" readonly class="form-control-plaintext">@_employee.BirthDate.ToShortDateString()</label>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label class="col-sm-5 col-form-label">Email</label>
                                <div class="col-sm-7">
                                    <label type="text" readonly class="form-control-plaintext">@_employee.Email</label>
                                </div>
                            </div>
                        </div>
                    </div>
                    <button type="button" class="btn btn-outline-primary" @onclick="@Close">Close</button>
                </div>
            </div>
        </div>
    </div>
}

```


