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
















