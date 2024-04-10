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

## LocalStorage

Для хранения состояния в LocalStorege можно применить пакет
Blazored LocalStorege

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

## Вызор Js из Blazor
OnAfterRenderAsync


## Создание Razor Class Library
Также можно сделать Lazy Loading

