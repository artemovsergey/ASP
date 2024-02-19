# ASP Core

## Паттерны проектирования

![](/Patterns.png)

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

# OnConfiguring

```Csharp
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=Myrtex;Trusted_Connection=True;");
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
using System.Linq.Dynamic.Core

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

# Application Services

```Csharp
public static class ApplicationServicesRegistration
{
    // Extension method for IServiceCollection
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        // Add logging services
        services.AddLogging();

        // Add MediatR services and register services from the current assembly
        services.AddMediatR(config => config.RegisterServicesFromAssemblies(
               Assembly.GetExecutingAssembly()));

        // Add validators from the assembly of CreateUserCommandValidator
        services.AddValidatorsFromAssembly(typeof(CreateUserCommandValidator).Assembly);

        // Add transient service for the validation pipeline
        services.AddTransient(
           typeof(IPipelineBehavior<,>),
           typeof(RequestValidationPipeline<,>));

        // Return the service collection
        return services;
    }
}
```

# OnModelCreating

```Csharp
 protected override void OnModelCreating(ModelBuilder modelBuilder)
 {
     // Configure indexes and relationships
     ConfigureEntityRelationships(modelBuilder);

     // Seed initial data
     SeedInitialData(modelBuilder);

     // Configure property conversions for User entity
     ConfigureUserPropertyConversions(modelBuilder);
 }

 // Configure indexes and relationships
 private void ConfigureEntityRelationships(ModelBuilder modelBuilder)
 {
     // Ensure Username is unique
     modelBuilder.Entity<User>().HasIndex(u => u.Username).IsUnique();

     // Configure one-to-many relationship between User and Record entities
     modelBuilder.Entity<Record>()
         .HasOne(p => p.User)
         .WithMany(u => u.Records)
         .HasForeignKey(p => p.UserId)
         .OnDelete(DeleteBehavior.Cascade);

     // Auto-include navigation property Records when querying User
     modelBuilder.Entity<User>()
         .Navigation(u => u.Records).AutoInclude();
 }

 // Seed initial data
 private void SeedInitialData(ModelBuilder modelBuilder)
 {
     var user = User.CreateUser(new Username("SuperAdmin2077CP"),
         new Password(HashPassword("qwerty28042002")),
         new Email("string@mail.com", true),
         Role.Admin,
         "ConfirmToken");

     var record = new Record
     {
         Id = Guid.NewGuid(),
         Title = "My day",
         Url = new Uri("https://www.youtube.com/"),
         DateCreated = DateTime.Now,
         DeadLine = DateTime.Now.AddMonths(1),
         Likes = 183,
         DisLikes = 13,
         IsPrivate = false,
         UserId = user.Id,
     };

     // Add initial data to User and Record entities
     modelBuilder.Entity<User>().HasData(user);
     modelBuilder.Entity<Record>().HasData(record);
 }

 // Hash a password
 private string HashPassword(string password)
 {
     var hashedBytes = SHA256.HashData(Encoding.UTF8.GetBytes(password));
     var hash = BitConverter.ToString(hashedBytes).Replace("-", "").ToLower();
     return hash;
 }

 // Configure property conversions for User entity
 private void ConfigureUserPropertyConversions(ModelBuilder modelBuilder)
 {
     // Configure conversion for Username property
     modelBuilder.Entity<User>()
         .Property(u => u.Username)
         .HasConversion(
             u => u.Value,
             u => new Username(u));

     // Configure conversion for Password property
     modelBuilder.Entity<User>()
         .Property(u => u.Password)
         .HasConversion(
             p => p.Value,
             p => new Password(p));

     // Configure conversion for Email property
     modelBuilder.Entity<User>()
         .Property(u => u.Email)
         .HasConversion(
             e => e.Value,
             e => new Email(e, true));
 }
```

# QRCode

```Csharp
public sealed class QRCodeGeneratorService : IQRCodeGeneratorService
{
    // QR code generator
    private readonly QRCodeGenerator _qrCodeGenerator;

    // Constructor
    public QRCodeGeneratorService()
    {
        _qrCodeGenerator = new QRCodeGenerator();
    }

    // Generate a QR code from a text string
    public string GenerateQRCodeFromText(string text)
    {
        try
        {
            if (string.IsNullOrEmpty(text))
                throw new ArgumentNullException(nameof(text), "Text cannot be null or empty.");

            // Create QR code data
            var data = _qrCodeGenerator.CreateQrCode(text, QRCodeGenerator.ECCLevel.Q);

            // Create bitmap from QR code data
            var bitmap = new BitmapByteQRCode(data);
            var qrCodeBytes = bitmap.GetGraphic(20);

            // Convert bitmap to image
            using var ms = new MemoryStream(qrCodeBytes);
            var qrCodeImage = new Bitmap(ms);

            // Convert image to base64 string
            using var msBase64 = new MemoryStream();
            qrCodeImage.Save(msBase64, ImageFormat.Png);
            var base64String = Convert.ToBase64String(msBase64.ToArray());

            return base64String;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            throw;
        }
    }
}
```

# Infrastructure Services

```Csharp
public static class InfrastructureServicesRegistration
{
    // Extension method for IServiceCollection
    public static IServiceCollection AddInfrastructureServices(this IServiceCollection services, IConfiguration configuration)
    {
        // Add DbContext to the services
        services.AddDbContext<AppDbContext>(opts =>
        {
            opts.UseSqlServer(configuration.GetConnectionString("Default") ??
                "Server=.; Database=PosteBin; Trusted_Connection=SSPI; Encrypt=Optional");
        });

        // Register repositories and services
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<IRecordRepository, RecordRepository>();
        services.AddScoped<IRecordCloudService, CloudService>();
        services.AddScoped<IQRCodeGeneratorService, QRCodeGeneratorService>();
        services.AddScoped<ITelegramService, TelegramService>();

        return services;
    }
}
```

# Configure Swagger for API documentation

```Csharp
builder.Services.AddSwaggerGen(c =>
{
    c.EnableAnnotations();
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "FP", Version = "v2077" });
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "Authorization using jwt token. Example: \"Bearer {token}\"",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey
    });
    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] { }
        }
    });
});
```

# Configure Serilog for logging
```Csharp
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration).CreateLogger();
builder.Host.UseSerilog();
```

# Configure authentication with JWT
```Csharp
builder.Services.AddAuthentication().AddJwtBearer(opts =>
{
    opts.SaveToken = true;
    opts.RequireHttpsMetadata = false;
    opts.TokenValidationParameters = new()
    {
        ValidateIssuer = false,
        ValidateAudience = false,
        ValidateIssuerSigningKey = true,
        ValidateLifetime = true,
        IssuerSigningKey =
           new SymmetricSecurityKey(
                Encoding.ASCII.GetBytes(
                    builder.Configuration["Jwt"] ?? throw new Exception("Jwt configuration not found. Please ensure it is set in the configuration file."))),
    };
});
```

# Configure authorization to require authenticated user
```Csharp
builder.Services.AddAuthorization(opts =>
{
    opts.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

# Add application and infrastructure services
```Csharp
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
```

# Configure middleware, CORS, authentication, and authorization
```Csharp
app.UseMiddleware<ExceptionHandlingMiddlwere>();
app.UseHttpsRedirection();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
```

# Middleware

```Csharp
public sealed class ExceptionHandlingMiddlwere : IMiddleware
{
    private readonly ILogger<ExceptionHandlingMiddlwere> _logger;

    public ExceptionHandlingMiddlwere(ILogger<ExceptionHandlingMiddlwere> logger) =>
        _logger = logger;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        try
        {
            await next(context);
        }
        catch (Exception ex)
        {

            _logger.LogError(ex, "Exception occurred: {message}", ex.Message);
            await HandleExeptionAsync(context, ex.Message);
        }
    }

    private async Task HandleExeptionAsync(HttpContext context, string exMassage)
    {
        HttpResponse response = context.Response;
        response.ContentType = "application/problem+json";

        var problemDetails = new ProblemDetails
        {
            Title = "An error occurred",
            Status = StatusCodes.Status500InternalServerError,
            Detail = exMassage,
            Instance = context.Request.Path
        };

        context.Response.StatusCode = StatusCodes.Status500InternalServerError;

        await response.WriteAsJsonAsync(problemDetails);
    }
```

# Serilog 

```json
"Serilog": {
  "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
  "MinimumLevel": "Debug",
  "WriteTo": [
    {
      "Name": "Console"
    },
    {
      "Name": "File",
      "Args": {
        "restrictedToMinimumLevel": "Information",
        "path": "Logs/infoLog-.txt",
        "rollingInterval": "Day"
      }
    },
    {
      "Name": "File",
      "Args": {
        "restrictedToMinimumLevel": "Error",
        "path": "Logs/errorLog-.txt",
        "rollingInterval": "Day"
      }
    }
  ],
  "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ]
},
```

# UserController Production

```Csharp
using Application.Features.Users;
using Application.Features.Users.Delete;
using Application.Features.Users.Get;
using Application.Features.Users.Update;
using Domain.Entities;
using Domain.IServices;
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Swashbuckle.AspNetCore.Annotations;
using System.Web;
using WebApi.Services;

namespace WebApi.Controllers;

[Route("api/users")]
[ApiController]
[Produces("application/json")]
public class UserController : ControllerBase
{
    private readonly IMediator _mediator;
    private readonly ITelegramService _telegramService;
    private readonly ILogger<UserController> _logger;

    public UserController(IMediator mediator, ILogger<UserController> logger, ITelegramService telegramService)
    {
        _mediator = mediator;
        _logger = logger;
        _telegramService = telegramService;
    }

    [Authorize(Roles = "Admin")]
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Обновляет пользователя по Id.")]
    [SwaggerResponse(StatusCodes.Status200OK, "User successfully updated")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "User not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> UpdateUserById(UpdateUserByIdCommand command)
    {
        _logger.LogInformation("Updating user by Id: {Id}", command.UserId);
        var response = await _mediator.Send(command);

        if (response.IsSuccess)
        {
            _logger.LogInformation("User with Id: {Id} successfully updated", command.UserId);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to update user with Id: {Id}. Reasons: {Reasons}", command.UserId, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpPut("me")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Обновляет текущего пользователя.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Current user successfully updated")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "Current user not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> UpdateCurrentUser(UpdateUserDto user)
    {
        var currentUser = HttpContext.User;
        var userId = UserServices.GetCurrentUserId(currentUser);

        var request = new UpdateUserByIdCommand
        {
            UserId = userId,
            Data = user
        };

        _logger.LogInformation("Updating current user Id: {Id}", userId);
        var response = await _mediator.Send(request);

        if (response.IsSuccess)
        {
            _logger.LogInformation("Current user Id: {Id} successfully updated", userId);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to update current user with ID: {Id}. Reasons: {Reasons}", userId, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpGet("{userId}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Получает пользователя по Id.")]
    [SwaggerResponse(StatusCodes.Status200OK, "User successfully retrieved")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "User not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> GetUserById(Guid userId)
    {
        var request = new FetchUserByIdRequest
        {
            UserId = userId,
        };

        _logger.LogInformation("Retrieving user by ID: {Id}", request.UserId);
        var response = await _mediator.Send(request);

        if (response.IsSuccess)
        {
            _logger.LogInformation("User with Id: {Id} successfully retrieved", response.Value.Id);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to retrieve user with Id: {Id}. Reasons: {Reasons}", response.Value.Id, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpGet("username/{username}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Получает пользователя по username.")]
    [SwaggerResponse(StatusCodes.Status200OK, "User successfully retrieved")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "User not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> GetUserByUsername(string username)
    {
        var request = new FetchUserByUsernameRequest { TargetUsername = username };

        _logger.LogInformation("Retrieving user by username: {username}", request.TargetUsername);
        var response = await _mediator.Send(request);

        if (response.IsSuccess)
        {
            _logger.LogInformation("User with username: {username} successfully retrieved", response.Value.Username);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to retrieve user with username: {username}. Reasons: {Reasons}", response.Value.Username, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Получает всех пользователей.")]
    [SwaggerResponse(StatusCodes.Status200OK, "All users successfully retrieved")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "Users not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> GetAllUsers([FromQuery] GetAllUsersRequest request)
    {
        _logger.LogInformation("Retrieving all users");
        var reponse = await _mediator.Send(request);

        if (reponse.IsSuccess)
        {
            _logger.LogInformation("All users successfully retrieved");
            return Ok(reponse.Value);
        }

        _logger.LogError("Failed to retrieve all users. Reasons: {Reasons}", reponse.Reasons);
        return NotFound(reponse.Reasons);
    }

    [Authorize(Roles = "Admin")]
    [HttpDelete]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Удаляет пользователя по Id.")]
    [SwaggerResponse(StatusCodes.Status200OK, "User successfully deleted")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "User not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> DeleteById(DeleteUsersByIdsCommand command)
    {
        _logger.LogInformation("Deleting user by ID: {Id}", command.UserIds);
        var response = await _mediator.Send(command);

        if (response.IsSuccess)
        {
            _logger.LogInformation("User with ID: {Id} successfully deleted", command.UserIds);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to delete user with ID: {Id}. Reasons: {Reasons}", command.UserIds, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "Admin")]
    [HttpDelete("me/{username}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Удаляет пользователя по Username.")]
    [SwaggerResponse(StatusCodes.Status200OK, "User successfully deleted")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "User not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> DeleteByUsername(string username)
    {
        var command = new DeleteByUsernameCommand { TargetUsername = username };

        _logger.LogInformation("Deleting user by Username: {Username}", command.TargetUsername);
        var response = await _mediator.Send(command);

        if (response.IsSuccess)
        {
            _logger.LogInformation("User with Username: {Username} successfully deleted", command.TargetUsername);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to delete user with Username: {Username}. Reasons: {Reasons}", command.TargetUsername, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpDelete("me")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Удаляет текущего пользователя.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Current user successfully deleted")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "Current user not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> DeleteCurrentUser()
    {
        var currentUser = HttpContext.User;
        var currentUserId = UserServices.GetCurrentUserId(currentUser);

        var command = new DeleteUsersByIdsCommand()
        {
            UserIds = new Guid[] { currentUserId },
        };

        _logger.LogInformation("Deleting current user by ID: {Id}", currentUserId);
        var response = await _mediator.Send(command);

        if (response.IsSuccess)
        {
            _logger.LogInformation("Current user with ID: {Id} successfully deleted", currentUserId);
            return Ok(response.Value);
        }

        _logger.LogError("Failed to delete current user with ID: {Id}. Reasons: {Reasons}", currentUserId, response.Reasons);
        return NotFound(response.Reasons);
    }

    [Authorize(Roles = "User, Admin")]
    [HttpPatch("confirm/{confirmToken}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Подтверждает электронную почту.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Mail confirmed", typeof(User))]
    [SwaggerResponse(StatusCodes.Status404NotFound, "Mail not confirmed", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> ConfirmEmail(string confirmToken)
    {
        var currentUser = HttpContext.User;
        var userId = UserServices.GetCurrentUserId(currentUser);

        string decodedToken = HttpUtility.UrlDecode(confirmToken);

        var command = new ConfirmUserEmailCommand
        {
            UserId = userId,
            ConfirmationToken = decodedToken
        };

        _logger.LogInformation("Confirming email for user by ID: {Id}", userId);
        var response = await _mediator.Send(command);
        if (response.IsSuccess)
        {
            _logger.LogInformation("Email for user with ID: {Id} successfully confirmed", userId);
            return Ok("Mail confirmed");
        }

        _logger.LogError("Failed to confirm email for user with ID: {Id}. Reasons: {Reasons}", userId, response.Reasons);
        return NotFound(response.Reasons);
    }
}

```

# Middleware

```Csharp
    public class HeadersMiddleware
    {
        private readonly RequestDelegate _next;
        public HeadersMiddleware(RequestDelegate next)
        {
            _next = next;
        }
        public async Task Invoke(HttpContext context)
        {
            context.Response.OnStarting(() =>
            {
                context.Response.Headers["X-Content-Type-Options"] =
                "nosniff";
                return Task.CompletedTask;
            });
            await _next(context);
        }
    }
}

/*
 ASP.NET Core создает промежуточное ПО только
один раз за весь жизненный цикл вашего приложения, поэтому
любые зависимости, внедряемые в  конструктор, должны иметь
жизненный цикл Singleton. Если вам нужно использовать ограниченные или временные зависимости с  жизненными циклами
Scoped или Transient, внедрите их в метод Invoke 
 */
```

