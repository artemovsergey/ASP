# Program.cs для API

```Csharp
using Microsoft.EntityFrameworkCore;
using WebAPILearning.Data;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

// если происходит цикл при связи 1:M через API
builder.Services.AddControllersWithViews()
    .AddNewtonsoftJson(options =>
    options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
);

/*
 * 
В ASP.NET Core службы (такие как контекст базы данных) должны быть зарегистрированы с помощью 
контейнера внедрения зависимостей. Контейнер предоставляет службу контроллерам.
 
 */

string connection = builder.Configuration.GetConnectionString("PostgreSQL");
builder.Services.AddDbContext<UserContext>(options => options.UseNpgsql(connection));

//builder.Services.AddDbContext<UserContext>(opt =>opt.UseInMemoryDatabase("UsersDatabase"));
// Добавляет контекст базы данных в контейнер внедрения зависимостей.
// Указывает, что контекст базы данных будет использовать базу данных в памяти.

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

# Настройка отображения Json

```Csharp
builder.Services.AddControllers()
 .AddJsonOptions(options =>
 {
 options.JsonSerializerOptions.WriteIndented = true;
 });
```













