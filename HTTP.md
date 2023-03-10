# HTTPClient

```Csharp
using Microsoft.Extensions.DependencyInjection;
using System.Net.Http;

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

}
```

# Json

```Csharp
app.MapGet("/", () => new Person() { Id=1,Name="user1"}); // auto serialize to json
```

```Csharp
app.MapGet("/{id?}", (int? id) =>
{
    if (id is null)
        return Results.BadRequest(new { Message = "Некорректные данные в запросе" });
    else if (id != 1)
        return Results.NotFound(new { Message = $"Объект с id={id} не существует" });
    else
        return Results.Json(new Person() { Id=2,Name="user2"});
});
```

















