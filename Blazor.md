# Blazor

# Режим рендеринга

```Csharp
builder.Services.AddRazorComponents()
    .AddInteractiveWebAssemblyComponents()
    .AddInteractiveServerComponents();
```

```Csharp
app.MapRazorComponents<App>()
    .AddInteractiveWebAssemblyRenderMode()
    .AddInteractiveServerRenderMode()
    .AddAdditionalAssemblies(typeof(ClientBlazorApp.Client._Imports).Assembly);
```

# Router


```razor
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <PageTitle>Not found</PageTitle>
        <LayoutView Layout="@typeof(MainLayout)">
            <p role="alert">Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```

Компонент Router представляет собой шаблонный компонент с двумя шаблонами. Шаблон Found
используется для известных маршрутов, а шаблон NotFound отображается, когда URI не
не соответствует ни одному из известных маршрутов. Вы можете заменить содержимое последнего, чтобы показать пользователю красивую
страницу ошибки пользователю.
Шаблон Found использует компонент RouteView, который отображает выбранный
компонент со своим макетом (или макетом по умолчанию). Когда компонент Router
инстанцируется, он будет искать в своем свойстве AppAssembly все компоненты, которые имеют атрибут
RouteAttribute (директива @page razor компилируется в RouteAttribute) и
выберет тот компонент, который соответствует URI текущего браузера. Например, компонент "Счетчик
имеет бритвенную директиву @page "/counter", и когда URL в браузере
совпадает с /counter, он отобразит компонент Counter в компоненте MainLayout.

Переведено с помощью DeepL.com (бесплатная версия)



# MainLayout

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

# _Imports.razor

```razor
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using static Microsoft.AspNetCore.Components.Web.RenderMode
```

# Компонент NavMenu

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

# Компонент App

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

# Применение автоматического рендеринга в компоненте

```
@rendermode InteractiveAuto
```

# Lambda

```Csharp
<button @onclick = "@(e => Show(e, buttonNumber)">
@code{
 
  private void Show(MouseEventArgs e, int ButtonNumber)
   {

   }
}
```

# Жизненынй цикл компонента

- Метод SetParametersAsync() устанавливает параметры компонента значениями, предоставленными родительским компонентом. В качестве параметра метод принимает объект типа ParameterView, который содержит набор значений для параметров компонента.
```Csharp
@code{
    string message = "Not set"; // значение по умолчанию, если для Password не передано значение
    [Parameter]
    public string? Password { get; set; }
    public override Task SetParametersAsync(ParameterView parameters)
    {
        // если в parameters есть Password
        if (parameters.TryGetValue<string>(nameof(Password), out var value))
        {
            if (value is null || value?.Length < 6)
            {
                message = "Password is invalid";
            }
            else
            {
                message = "Password is strong";
            }
        }
        return base.SetParametersAsync(parameters);
    }
}
```

- Методы OnInitialized/OnInitializedAsync вызываются после инициализации компонента после получения данных для параметров в методе SetParametersAsync. При синхронной инициализации инциализация родительского компонента гарантированно завершится первой. При асинхронной инициализации порядок завершения инициализации родительского и дочернего компонентов невозможно определить, так как он зависит от выполняемого кода инициализации.

- Методы OnParametersSet/OnParametersSetAsync вызываются после инициализации компонента в OnInitialized()/OnInitializedAsync(), а также после повторного рендеринга родительского компонента. На этом этапе значения параметров установлены, мы их можем использовать для некоторой обработки или даже изменить.

- Методы OnAfterRender()/OnAfterRenderAsync() вызываются после рендеринга компонента. Здесь можно выполнить какую-то дополнительную логику инициализации с использованием содержимого компонента. В качестве параметра эти методы получают булевое значение - если оно равно true, то рендеринг компонента произведен первый раз.
Стоит отметить, что методы OnAfterRender()/OnAfterRenderAsync() не вызываются во временя пререндеринга на сервере. Эти методы вызываются уже после того, как скрипт Blazor (blazor.web.js) запустится в браузере, а компонент будет перезапущен в интерактивном режиме рендеринга.

# Передача события из дочернего компонента в родительский

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

# Динамические компоненты

RenderFragment @ChildContent
DynamicComponent

# Обработка ошибок

```
<ErrorBoundary>
	<ChildContent>
 	
	</ChildContent>

        <ErrorContent>

        </ErrorContent>
</ErrorBoundary>
```

# Настройка клиента Http в API 

```Csharp
builder.Services.AddHttpClient<IServiceName, ServiceName>(c => c.BaseAddress = new Uri(builder.HostEnviroment.BaseAddress))
```

# Инкапсуляция вызовов API

Так можно отдельно вынести получение и отправку данных данных через API в методы

```Csharp
public async Task<IEnumerable<Employee>> GetAllEmployees(){

return await JsonSerializer.DeserializeAsync<IEnumerable<Employee>> (await _httpClien.GetStreamAsync(), new JsonSerializerOption() { PropertyNameCaseInsensitive = true } );
}
```

#  LocalStorage

using Blazored LocalStorage

Для хранения состояния в LocalStorege можно применить пакет
Blazored LocalStorage

```Csharp
@inject Blazored.LocalStorage.ILocalStorageService localStorage
var firstName = await localStorage.GetItemAsync<string>("EmployeeFirstName");
```

- SetItem()
- GetItem()
- ContainKey()
- RemoveItem()

# Валидация DataAnnotation

- DataAnnotation
- DataAnnotationValidator
- ValidationSummary

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

Пользовательский валидатор:
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

# FluentValidation

В проекте домена надо поставить ```FluetnValidation```.

На клиенте в Blazor надо:

- зарегистрировать класс в контейнере.
- применить атрибут в форме ```<FluentValidationValidator/>```
- установить пакет Blazored.FluentValidation
  

Domen/Validations/LoginValidator.cs
```Csharp
public class LoginValidator : AbstractValidator<User>
{
	private readonly HttpClient _httpClient;

	public LoginValidator(HttpClient httpClient)
	{
		_httpClient = httpClient;

		RuleFor(m => m.Login)
			.MustAsync(async (name, cancellation) => await IsNameValidAsync(name))
			.WithMessage("Имя уже используется.");
	}

	private async Task<bool> IsNameValidAsync(string name)
	{
		var response = await _httpClient.PostAsJsonAsync("https://localhost:7214/api/Users/checklogin", name);
		Console.WriteLine(response.IsSuccessStatusCode);

		if (!response.IsSuccessStatusCode)
		{
			// Попытка прочитать сообщение об ошибке из ответа
			var errorContent = await response.Content.ReadAsStringAsync();
			Console.WriteLine($"Ошибка: {errorContent}");
			return false;
		}

		return response.IsSuccessStatusCode;
	}
}
```

# Вызор Js из Blazor

OnAfterRenderAsync


# Создание Razor Class Library

Также можно сделать Lazy Loading

```Csharp
@using System.Reflection
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader
<Router AppAssembly="@typeof(Program).Assembly"
 AdditionalAssemblies="@additionalAssemblies"
 OnNavigateAsync="OnNavigate">
 <Found Context="routeData">
 <RouteView RouteData="@routeData" DefaultLayout="@typeof
(MainLayout)" />
 </Found>
 <NotFound>
 <LayoutView Layout="@typeof(MainLayout)">
<p>Sorry, there's nothing at this address.</p>
 </LayoutView>
 </NotFound>
<Navigating>
 Loading additional components...
 </Navigating>
</Router>
@code {
 private List<Assembly> additionalAssemblies =
 new List<Assembly>
 {
 };
 private async Task OnNavigate(NavigationContext context)
 {
 if( context.Path == "counter")
 {
 var assembliesToLoad = new List<string>
 {
 "LazyLoading.Library.dll"
 };
 var assemblies = await assemblyLoader.LoadAssembliesAsync
(assembliesToLoad);
 additionalAssemblies.AddRange(assemblies);
 }
 }
}
```

```
builder.Services.AddScoped<LazyAssemblyLoader>();
```

 Если ресурс находится в проекте приложения Blazor
проект приложения, путь начинается с папки wwwroot, но для библиотечных проектов
URL должен начинаться с _content/{LibraryProjectName} и ссылаться на папку wwwroot
папку вашего библиотечного проекта.

```
<img src="_content/Components.Library/cloud.png" alt="Cloud"/>
```

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

# Logger

```Csharp
    @inject ILoggerFactory LoggerFactory
    protected override async Task OnInitializedAsync()
    {
        _logger = LoggerFactory.CreateLogger<Auth>();
    }

```

# Компонент ImputSelect

```Csharp
    <InputSelect @bind-Value="@user.RoleId" multiple>
	@foreach (var r in Roles)
	{
	    <option value="@r.Id">@r.Name</option>
	}
    </InputSelect>
```

# Подключение css в компонентах

- создать файл Name.razor.css
- подключить в index файл имя_проекта.styles.css

```
 <link href="SportStore.Blazor.styles.css" rel="stylesheet" type="text/css">
```

# События

Если элемент html имеет атрибуты вида on{СОБЫТИЕ}, которые позволяют связать событие с некоторой функцией javascript (например, атрибут onclick), то Blazor предоставляет их двойники - атрибуты типа @on{СОБЫТИЕ}, которые позвляют прикрепить к событию в качестве обработчика метод компонента.

Табилца событий https://metanit.com/sharp/blazor/2.5.php

# Каскадные значения

```Csharp
<CascadingValue Value="@now">
    <Main />
</CascadingValue>
 
@code {
    DateTime now = DateTime.Now;
}
```

```Csharp
<h2>Date: @DateTime?.ToShortDateString()</h2>
<Time />
 
@code {
 
    [CascadingParameter]
    DateTime? DateTime { get; set; }
}
```

# Использование ссылки на компонент ref

```Csharp
<Alert @bind-Show="Show" @ref="alert" >
    <span class="oi oi-check mr-2" aria-hidden="true"></span>
    <strong>Blazor is so cool!</strong>
</Alert>
<button @onclick="@(() => alert.Dismiss())"> Закрыть компонент Alert</button>

@code{
    public bool Show { get; set; } = true;
    private Alert alert { get; set; }
}
```

# VirtualizeComponent

Компонент для получения больших объекмов данных в интерфейс. Также поддерживает пагинацию

# IoC

Blazor WebAssembly экземпляры Scoped по умолчанию привязаны к вкладке браузера (приложению
приложения); они ведут себя как синглтоны, так что разницы нет.

```
@implements IDisposable
@code {
 public void Dispose()
 => transientService.Dispose();
}
```

Переходные объекты всегда разные; новый экземпляр предоставляется каждому компоненту
и каждому сервису.
Скопированные объекты одинаковы для подключения пользователя, но различны для разных
пользователей и соединений. Вы можете получить производные от класса OwningComponentBase, если вам нужно
поведение для конкретного компонента.
Объекты-синглтоны одинаковы для каждого объекта и каждого запроса, но все же имеют
разное время жизни в Blazor WebAssembly и Blazor Server.

# Task and ValueTask

# Настройка сериализации

Каждый из этих методов принимает необязательный параметр ```JsonSerializerOptions```, который позволяет вам
управлять процессом сериализации JSON. Например, опции по умолчанию будут сериализовать
имена свойств с сохранением регистра имени свойства. Однако есть сервисы, которые
требуют использовать верблюжье написание для свойств.

Чтобы изменить регистр, можно установить свойство ```PropertyNamingPolicy```.
Здесь мы установили значение ```JsonNamingPolicy.CamelCase```. Этот пример также показывает, как можно
управлять сериализацией перечислений. Обычно перечисления сериализуются с
их значение int. Например, Spiciness.Spicy будет сериализовано как 1. Но если вы хотите, вы можете
можно также использовать имя значения перечисления, так что Spiciness.Spicy будет сериализовано
как "Spicy". Для этого используйте конвертер JsonStringEnumConverter. Не
забывайте, что вам придется передать JsonSerializerOptions в качестве дополнительного аргумента с помощью функции
GetFromJsonAsync и аналогичных методов.

# Loader

```
<div class="mx-auto text-center mb-3 mt-3">
 <div class="spinner-border text-danger" role="status">
 <span class="visually-hidden">Loading...</span>
 </div>
</div>
```

# Alert Component

```Csharp
@if (Show)
{

    <div class="modal show" tabindex="-1" role="dialog" style="display:block">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Modal title</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="modal-body">
                <p>Modal body text goes here.</p>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-primary">Save changes</button>
                <button type="button" class="btn btn-secondary" data-dismiss="modal" @onclick="() => Close()">Close</button>
            </div>
        </div>
    </div>
    </div>


}
else
{
    <h3> Show: @Show </h3>
}

@code {

    [Parameter]
    public bool Show { get; set; }

    [Parameter]
    public EventCallback<bool> ShowChanged { get; set; }

    private async Task OnShowChanged(bool newValue)
    {
        Show = newValue;
        await ShowChanged.InvokeAsync(Show);
    }

    public void Open()
    {
        Show = true;
      
    }

    public void Close()
    {
        Show = false;
        //StateHasChanged();
    }
}
```

## InputChange

```
<input type="number"
 value="@increment"
 @onchange="@((ChangeEventArgs e)
 => increment = int.Parse($"{e.Value}"))" />
```

## Опции

- Preventing Default Actions - отмена действий браузера по умолчанию
- Stopping Event Propagation - отмена действий событий вверх по дереву

# Format
- форматирование даты  <input @bind="@Today" @bind:format="yyyy-MM-dd" />

# Обновление интерфейса StateHasChange

Blazor будет повторно отображать страницу при каждом возникновении события. Он также будет повторно отображать страница в случае асинхронных операций. Однако некоторые изменения не могут быть обнаружены.

# Таймер

```Csharp
private void AutoIncrement()
{
 var timer = new System.Threading.Timer(
 callback: (_) => { IncrementCount(); StateHasChanged(); },
 state: null,
 dueTime: TimeSpan.FromSeconds(1),
 period: TimeSpan.FromSeconds(1));
}

StateHasChanged()
```

# Управление состоянием 

- FLuxor

Мы можем хранить состояние нашего приложения в локальном хранилище,
на сервере и в URL. Затем мы рассмотрели паттерн redux, который используется для создания
сложных приложений. Redux упрощает эту задачу, применяя несколько принципов.
Данные компонентов привязываются к объекту Store, который вы изменяете путем диспетчеризации действий
которые содержат требуемое изменение. Затем редуктор применяет это изменение к хранилищу, что
что вызовет обновление ваших компонентов, завершая круг. Redux и flux имеют
преимущество в том, что в итоге вы получаете множество маленьких классов, которые легче поддерживать,
применяя принцип единой ответственности.

flux у нас несколько хранилищ, в то время как redux предпочитает поместить все в одно хранилище.

# SignalR

Базовый класс Hub имеет свойство Clients типа IHubCallerClients. Этот интерфейс
имеет три свойства: All, Caller и Others. Свойство All предоставляет доступ ко
всем клиентам, подключенным к концентратору, свойство Caller возвращает клиента, вызывающего концентратор.
BoardHub, а свойство Others возвращает всех клиентов, кроме вызывающего. 

# gRPC

До появления SOAP и REST разработчики использовали вызовы удаленных процедур (Remote Procedure Calls, RPC) для вызова методов другого процесса.
Вызовы удаленных процедур (RPC) для вызова методов в другом процессе. Мы уже видели, как взаимодействовать
с помощью REST API между двумя приложениями, но сериализация в JSON и обратно
приводит к некоторым накладным расходам. В основном это делается для того, чтобы сделать сообщения человекочитаемыми. Если
коммуникация должна происходить только между приложениями и нет необходимости в
человекочитаемой форме, мы можем использовать gRPC. Поскольку сериализованные данные не обязательно должны
быть читаемыми человеком, они могут быть более компактными и эффективными, а значит, и более производительными.

Что такое gRPC? Этот фреймворк предоставляет нам современный и высокоэффективный способ
взаимодействия с использованием тех же принципов RPC. Он работает для таких языков, как C#,
Java, JavaScript, Go, Swift, C++, Python, Node.js и других языков. Он обеспечивает
совместимость между различными языками благодаря использованию языка определения интерфейсов
Язык определения интерфейсов (IDL), описанный в файлах .proto. Эти файлы затем используются для генерации
необходимый код, используемый как сервером, так и клиентом.
Использование gRPC отличается высокой производительностью и малым весом. Вызов gRPC может быть в восемь
раз быстрее, чем эквивалентный вызов REST. Поскольку в нем используется бинарная сериализация, сообщения
могут быть на 60-80 % меньше, чем JSON.


# Заметки

- методика изучения Blazor: привязка, события, валидация
- попробовать обработчики писать через лямбда функции
- альтернативная техника для атрибута @bind
- огранизовать работу отдельно с razor, cs, css
- сделать компонент без интерфейса
