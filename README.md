## ASP.Net Core 6 ( > 2016 )

### Виды приложений на ASP

Можно создать пустое приложение
Можно создать веб-приложение
Можно создать веб-приложение MVC
Можно создать API веб-приложение



Форма
```asp
          <form asp-action="rsvpform" methdod="post">
                
                <p>
                    <label asp-for="Name">Your Name</label>
                    <input asp-for="Name"/>
                </p>

                <p>
                    <label asp-for="Email">Ваша почта</label>
                    <input asp-for="Email"/>
                </p>

                <p>
                    <label asp-for="Phone">Телефон</label>
                    <input asp-for="Phone"/>
                </p>
                <p>
                    <label> WillAttend</label>
                    <select asp-for="WillAttend">
                        <option value="Choose an option"></option>
                        <option value="true">Да</option>
                        <option value="false">Нет</option>
                    </select>
                </p>
                <button type="submit">Отправить</button>

            </form>
```

Каждый элемент ассоциирован со свойством модели с помощью дескриптора asp-for

Все почти также как в ```Rails```

- Bootstrap идет из коробки
- Подключение css и javascript удобное
- Есть также свои дескрипторы для тегов
- Есть валидации в моделях
- Аналог params, который надо изучить
- Есть соглашения по рендерингу
- Роутинг еще не смотрел
- Тестирвоание через TDD еще не смотрел
- Настройку конфигурации еще не смотрел
- Подключение Entity Framework еще не смотрел
- Есть частичные шаблоны
- Надо посмотреть сесиии
- Развертывание как делать

---
Цель паттерна MVC - разделение приложения на три функциональных области, каждая из которых может содержать и логиу и данные. Цель не в том, чтобы устранить логику из модели. Наоборот, цель в том, чтобы гарантировать наличие в модели только логики предназначенной для создания и управления данными


## Конфигурационный файл для пустого проекта для подключения MVC

```ASP

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

var app = builder.Build();


// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}


//app.MapGet("/", () => "Hello World!");

app.UseDeveloperExceptionPage(); // Ошибки при отладке
app.UseHttpsRedirection();
app.UseStaticFiles(); // доставка статического содержимого

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();


```
## Null условнаяя проверка с объединением
```asp

string name = р?.Name ?? "No name"; // null условная операция с объединением

```

## Автоматические реализуемые свойства

```asp
puЫic string Nаш.е { get; set; } 

```
## Сопоставление с образцом

```asp

if (data is decimal d) { }

```
Замечание: сопоставлнеие с образцом также работает с оператором ```switch```

---

## Применение лямбда выражений

```asp
pubclic ViewResult Index() {
return View()
}

```asp
pubclic ViewResult Index() => View()
}

```

## Передача данных из контроллера в представление через ViewBag

В контроллере метода можно прописать
``` ViewBag.AnyProperty = 2;```

В представлении можно:

```
@ViewBag.AnyProperty
```
## Возможности механизма представления Razor

Можно добавлять значения в аттрибуты элементов разметки
Условные операторы if, switch
Циклы foreach


## Подключение Bootstrap и js

wwwroot папка
В ней должны быть ресурсы

В шаблоне сделать ссылку
```html
<link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.css"/>
<script src = "js/"></script>
```

## Настройка Postman для localhost

Finally fixed it by turning off File > Settings > General > SSL Certificate Verification

## Bundler and Minifier
Установить расширение

## Надо потом изучить
интерфейсы в С#
тестирование моделей, контроллеров, представлений через xUnit

## Порядок работы над новым проектом
1.Создать пустой проект ASP.NET Core 6

2.Создать папки Models,Controllers,Views

3.Скопировать конфигурационный конфиг по MVC паттерн в Program.cs

4.В папке Views создадим импортируемый шаблон для пространства имен _ViewImports.cshtml
```asp
@using SportsStore.Models // название приложения ? проверить 
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers 

```

5.Создать проект модульного тестирования xUnit

6.Добавить в ссылки проект и Moq. Это можно сделать через файл конфигурации ```asp <PackageReference Include="Moq" Version="4.7.99" /> ``` или через NuGet

7.Создаем модель предметной области. В папке Models создаем файл Product.cs














