# HTTPClient

```Csharp
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Net.Http;
using System.Net.Http.Json;

namespace HTTPConsole;

internal class Program
{
    static HttpClient? httpClient;

    static async Task Main(string[] args)
    {
        var socketsHandler = new SocketsHttpHandler
        {
            PooledConnectionLifetime = TimeSpan.FromMinutes(2)
        };
        httpClient = new HttpClient(socketsHandler);

        // использование HttpClient

        // определяем данные запроса
        using HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, "https://metanit.com/sharp/net/2.2.php");
        // выполняем запрос
        HttpResponseMessage response = await httpClient.SendAsync(request);


        // просматриваем данные ответа
        // статус
        Console.WriteLine($"Status: {response.StatusCode}\n");
        //заголовки
        Console.WriteLine("Headers");
        foreach (var header in response.Headers)
        {
            Console.Write($"{header.Key}:");
            foreach (var headerValue in header.Value)
            {
                Console.WriteLine(headerValue);
            }
        }

        // содержимое ответа
        Console.WriteLine("\nContent");
        string content = await response.Content.ReadAsStringAsync();
        Console.WriteLine(content);

        //

        // получаем ответ

        Console.WriteLine("------------------------------------------");
        using HttpResponseMessage response1 = await httpClient.GetAsync("https://www.google.com");
        // получаем ответ
        string content2 = await response.Content.ReadAsStringAsync();
        Console.WriteLine(content2);

        Console.WriteLine("------------------------------------------");

        try
        {
            object? data = await httpClient.GetFromJsonAsync("https://localhost:7009/1", typeof(Person));
            if (data is Person person)
            {
                Console.WriteLine($"Name: {person.Name} ");
            }
        }
        catch(Exception ex) 
        {
           Console.WriteLine(ex.Message);
        }

        Console.WriteLine("------------------------------------------");
        
        HttpResponseMessage response2 = await httpClient.GetAsync("https://localhost:7009/1");
        // если запрос завершился успешно, получаем объект Person
        Person? person1 = await response2.Content.ReadFromJsonAsync<Person>();
        Console.WriteLine($"Name: {person1?.Name}");


        Console.WriteLine("------------------------------------------");

        StringContent content1 = new StringContent("Tom");
        // определяем данные запроса
        using var request1 = new HttpRequestMessage(HttpMethod.Post, "https://localhost:7009/data");
        // установка отправляемого содержимого
        request1.Content = content1;

        // отправляем запрос
        using var response3 = await httpClient.SendAsync(request1);
        // получаем ответ
        string responseText = await response3.Content.ReadAsStringAsync();
        Console.WriteLine(responseText);


        Console.WriteLine("------------------------------------------");
        // Использование методов PostAsync/PactchAsync/PutAsync позволяет несколько упростить код

        StringContent content3 = new StringContent("Tom");
        using var response4 = await httpClient.PostAsync("https://localhost:7009/data", content3);
        string responseText1 = await response4.Content.ReadAsStringAsync();
        Console.WriteLine(responseText1);

        // отправка json
        Console.WriteLine("------------------------------------------");


        // отправляемый объект 
        Person tom = new Person { Name = "Tom" };
        // создаем JsonContent
        JsonContent content4 = JsonContent.Create(tom);
        // отправляем запрос
        using var response5 = await httpClient.PostAsync("https://localhost:7009/create", content4);
        Person? person3 = await response5.Content.ReadFromJsonAsync<Person>();
        Console.WriteLine($"{person3?.Id} - {person3?.Name}");


        // упрощенме post,put,patch
        Console.WriteLine("------------------------------------------");

        // отправляемый объект 
        Person tom1 = new Person {Name="Tom"};
        // отправляем запрос
        using var response6 = await httpClient.PostAsJsonAsync("https://localhost:7009/create", tom1);
        Person? person4 = await response6.Content.ReadFromJsonAsync<Person>();
        Console.WriteLine($"{person4?.Id} - {person4?.Name}");


        Console.ReadLine();

    }

    public void TestConnectionHttp()
    {

        // определяем коллекцию сервисов
        var services = new ServiceCollection();
        // добавляем сервисы, связанные с HttpClient, в том числе IHttpClientFactory
        services.AddHttpClient();
        // создаем провайдер сервисов
        var serviceProvider = services.BuildServiceProvider();
        // получаем сервис IHttpClientFactory
        var httpClientFactory = serviceProvider.GetService<IHttpClientFactory>();
        // создаем объект HttpClient
        var httpClient = httpClientFactory?.CreateClient();

        // использование HttpClient
    }

    public class Person
    {
        public string Id { get; set; }
        public string Name { get; set; }
    }


}
```

# Json from Program.cs

```Csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

//app.MapGet("/", () => new Person() { Id=1,Name="user1"}); // auto serialize to json

app.MapGet("/{id?}", (int? id) =>
{
    if (id is null)
        return Results.BadRequest(new { Message = "Некорректные данные в запросе" });
    else if (id != 1)
        return Results.NotFound(new { Message = $"Объект с id={id} не существует" });
    else
        return Results.Json(new Person() { Id="2",Name="user2"}); 
        // в общем виде, когда не нужна дополнительная информация в ответе
});


app.MapPost("/data", async (HttpContext httpContext) =>
{
    using StreamReader reader = new StreamReader(httpContext.Request.Body);
    string name = await reader.ReadToEndAsync();
    return $"Получены данные: {name}";
});

app.MapPost("/create", (Person person) =>
{
    // устанавливает id у объекта Person
    person.Id = Guid.NewGuid().ToString();
                     // отправляем обратно объект Person

    //return Results.Json(person);
    return person;
});


app.Run();

public class Person
{
    public string Id { get; set; }
    public string Name { get; set; }
}
```



# Json

```Csharp

        Console.WriteLine("------------------------------------------");

        try
        {
            object? data = await httpClient.GetFromJsonAsync("https://localhost:7009/1", typeof(Person));
            if (data is Person person)
            {
                Console.WriteLine($"Name: {person.Name} ");
            }
        }
        catch(Exception ex) 
        {
           Console.WriteLine(ex.Message);
        }

        Console.WriteLine("------------------------------------------");
        
        HttpResponseMessage response2 = await httpClient.GetAsync("https://localhost:7009/1");
        // если запрос завершился успешно, получаем объект Person
        Person? person1 = await response2.Content.ReadFromJsonAsync<Person>();
        Console.WriteLine($"Name: {person1?.Name}");
        
```

# API Endpoint for User

```Csharp
int id = 1; // для генерации id объектов
// начальные данные
List<Person> users = new List<Person>
{
    new() { Id = id++, Name = "Tom", Age = 37 },
    new() { Id = id++, Name = "Bob", Age = 41 },
    new() { Id = id++, Name = "Sam", Age = 24 }
};
 
var builder = WebApplication.CreateBuilder();
var app = builder.Build();
 
app.MapGet("/api/users", () => users);
 
app.MapGet("/api/users/{id}", (int id) =>
{
    // получаем пользователя по id
    Person? user = users.FirstOrDefault(u => u.Id == id);
    // если не найден, отправляем статусный код и сообщение об ошибке
    if (user == null) return Results.NotFound(new { message = "Пользователь не найден" });
 
    // если пользователь найден, отправляем его
    return Results.Json(user);
});
 
app.MapDelete("/api/users/{id}", (int id) =>
{
    // получаем пользователя по id
    Person? user = users.FirstOrDefault(u => u.Id == id);
 
    // если не найден, отправляем статусный код и сообщение об ошибке
    if (user == null) return Results.NotFound(new { message = "Пользователь не найден" });
 
    // если пользователь найден, удаляем его
    users.Remove(user);
    return Results.Json(user);
});
 
app.MapPost("/api/users", (Person user) =>
{
 
    // устанавливаем id для нового пользователя
    user.Id = id++;
    // добавляем пользователя в список
    users.Add(user);
    return user;
});
 
app.MapPut("/api/users", (Person userData) =>
{
 
    // получаем пользователя по id
    var user = users.FirstOrDefault(u => u.Id == userData.Id);
    // если не найден, отправляем статусный код и сообщение об ошибке
    if (user == null) return Results.NotFound(new { message = "Пользователь не найден" });
    // если пользователь найден, изменяем его данные и отправляем обратно клиенту
 
    user.Age = userData.Age;
    user.Name = userData.Name;
    return Results.Json(user);
});
 
app.Run();
 
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public int Age { get; set; }
}
```

# Определение клиента на ASP Core

## GET-запрос

```Csharp
    List<Person>? people = await httpClient.GetFromJsonAsync<List<Person>> ("https://localhost:7094/api/users");
    if (people != null)
    {
        foreach (var person in people)
        {
            Console.WriteLine(person.Name);
        }
    }
```

## Получение одного объекта

```Csharp
int id = 1; // получаем объект с id=1
Person? person = await httpClient.GetFromJsonAsync<Person>($"https://localhost:7094/api/users/{id}");
```

## Добавление данных

```Csharp
// отправляемый объект
var mike = new Person { Name = "Mike", Age = 31 };
using var response = await httpClient.PostAsJsonAsync("https://localhost:7094/api/users/", mike);
// считываем ответ и десериализуем данные в объект Person
Person? person = await response.Content.ReadFromJsonAsync<Person>();
Console.WriteLine($"{person?.Id} - {person?.Name}");
```

## Изменение данных

```Csharp
// id изменяемого объекта
int id = 1;
// отправляемый объект
var tom = new Person { Id = id, Name = "Tomas", Age = 38 };
using var response = await httpClient.PutAsJsonAsync("https://localhost:7094/api/users/", tom);
if (response.StatusCode == System.Net.HttpStatusCode.NotFound)
{
    // если возникла ошибка, считываем сообщение об ошибке
    Error? error = await response.Content.ReadFromJsonAsync<Error>();
    Console.WriteLine(error?.Message);
}
else if (response.StatusCode == System.Net.HttpStatusCode.OK)
{
    // десериализуем ответ в объект Person
    Person? person = await response.Content.ReadFromJsonAsync<Person>();
    Console.WriteLine($"{person?.Id} - {person?.Name} ({person?.Age})");
}
```
## Удаление данных

```Csharp
// id удаляемого объекта
int id = 1;
Person? person = await httpClient.DeleteFromJsonAsync<Person>($"https://localhost:7094/api/users/{id}");
Console.WriteLine($"{person?.Id} - {person?.Name} ({person?.Age})");
```




