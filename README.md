# ASP Core

# Project.Domen

- Models
- Validations

```xml
  <ItemGroup>
	  <PackageReference Include="FluentValidation" Version="11.9.1" />
  </ItemGroup>
```

- Валидация
```Csharp
public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(x => x.Name).Length(5).WithMessage("Please enter a name");
        RuleFor(x => x.Surname).NotEmpty().WithMessage("Please enter a description");
        RuleFor(x => x.Password).NotEmpty().WithMessage("Please enter a location");
        //RuleFor(x => x.Patronymic).GreaterThan(0).WithMessage("Please enter a length");
        RuleFor(x => x.Login).NotEmpty().WithMessage("Please add a route instruction");
        RuleForEach(x => x.Items).SetValidator(new ItemValidator());
        RuleFor(x => x.Items).NotEmpty().WithMessage("Please add a route instruction");
        RuleFor(x => x.Waypoints).NotEmpty().WithMessage("Please add a waypoint");
        Console.WriteLine("Работает валидатор UserValidator!");
    }
}
```

# Project.Infrastructure

- Configurations
- Migrations
- Data/Context.cs

- Пакеты
```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.4" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.4">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.4" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.4">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
```

- Сервисы
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


- Конфигурация
```Csharp
public class UserConfig : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.Property(x => x.Name).IsRequired();
        builder.Property(x => x.Description).IsRequired();
        builder.Property(x => x.Location).IsRequired();
        builder.Property(x => x.TimeInMinutes).IsRequired();
        builder.Property(x => x.Length).IsRequired();
    }
}
```

- Контекст
```Csharp
public  class SportStoreContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Role> Role { get; set; }
    public DbSet<Item> Items { get; set; }

    public SportStoreContext()
    {
        //Database.EnsureDeleted();
        //Database.EnsureCreated();    
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=SportStoreBlazor;Trusted_Connection=True;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        //modelBuilder.ApplyConfiguration(new UserConfig());
    }
}
```
```
/// <summary>
/// Клаас для работы автогенерации restAPI черезе IDE по модели и контексту
/// если конструктор с параметрами
/// </summary>
public class ExampleContextFactory : IDesignTimeDbContextFactory<ExampleContext>
{
    public ExampleContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<ExampleContext>();
        optionsBuilder.UseNpgsql("Host=localhost;Port=5432;Database=Example;Username=postgres;Password=root");

        return new ExampleContext(optionsBuilder.Options);
    }
}
```


# Project.Application

Структура
- Interfaces
- Repositories
- Handlers
- Requests
- ApplicationService.cs


Пакеты
- Mediatr


Сервисы
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

        return services;
    }
}
```

Request
```Csharp
public record AddUserRequest(User user) : IRequest<AddUserRequest.Response>
{
    public const string RouteTemplate = "api/users";
    public record Response(int userId);
}

public class AddUserRequestValidator : AbstractValidator<AddUserRequest>
{
    public AddUserRequestValidator()
    {
        RuleFor(x => x.user).SetValidator(new UserValidator());
    }
}
```

Handler
```Csharp
public class AddUserHandler : IRequestHandler<AddUserRequest,AddUserRequest.Response>
{
    private readonly HttpClient _httpClient;
    private readonly string BaseUrl = "https://localhost:7214";
    public AddUserHandler(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    public async Task<AddUserRequest.Response> Handle(AddUserRequest request, CancellationToken cancellationToken)
    {
        Console.WriteLine("Работает метод Handle из Handler");
        var response = await _httpClient.PostAsJsonAsync<AddUserRequest>($"{BaseUrl}/{AddUserRequest.RouteTemplate}", request, cancellationToken);
        if (response.IsSuccessStatusCode)
        {
            var userId = await response.Content.ReadFromJsonAsync<int>(cancellationToken:cancellationToken);
            return new AddUserRequest.Response(userId);
        }
        else
        {
            return new AddUserRequest.Response(-1);
        }
    }
}
```

Пример загрузки изображения

```Csharp
public record UploadUserImageRequest(int UserId, IBrowserFile File) : IRequest<UploadUserImageRequest.Response>
{
    public const string RouteTemplate = "/api/Users/{UserId}/images";
    public record Response(string ImageName);
}
```

```Csharp
public class UploadUserImageHandler : IRequestHandler<UploadUserImageRequest, UploadUserImageRequest.Response>
{
    private readonly HttpClient _httpClient;
    public UploadUserImageHandler(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    public async Task<UploadUserImageRequest.Response> Handle(UploadUserImageRequest request, CancellationToken cancellationToken)
    {
        var fileContent = request.File.OpenReadStream(request.File.Size, cancellationToken);

        using var content = new MultipartFormDataContent();
        content.Add(new StreamContent(fileContent), "image", request.File.Name);
        var response = await _httpClient.PostAsync(UploadUserImageRequest.RouteTemplate.Replace("{UserId}", request.UserId.ToString()), content, cancellationToken);
        if (response.IsSuccessStatusCode)
        {
            var fileName = await response.Content.ReadAsStringAsync(cancellationToken: cancellationToken);
            return new UploadUserImageRequest.Response(fileName);
        }
        else
        {
            return new UploadUserImageRequest.Response("");
        }
    }
}
```

# Project.API

Структура проекта:
- ApiEndPoints
- Controllers
- Hubs
- Images
- Services

Пакеты:
```xml
  <ItemGroup>
    <PackageReference Include="Ardalis.ApiEndpoints" Version="4.1.0" />
	<PackageReference Include="MediatR" Version="12.3.0" />
    <PackageReference Include="Google.Protobuf" Version="3.26.1" />
    <PackageReference Include="Grpc.AspNetCore" Version="2.62.0" />
    <PackageReference Include="Grpc.AspNetCore.Web" Version="2.62.0" />
    <PackageReference Include="Grpc.Net.Client" Version="2.62.0" />
    <PackageReference Include="Grpc.Net.Client.Web" Version="2.62.0" />
    <PackageReference Include="Grpc.Tools" Version="2.62.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly.Server" Version="8.0.6" />
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.4" />
    <PackageReference Include="SixLabors.ImageSharp" Version="3.1.4" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.4.0" />
    <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
  </ItemGroup>
```

Program
```Csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddControllers().AddJsonOptions(x => x.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles);         
builder.Services.AddScoped<SportStoreContext>();
builder.Services.AddScoped<UserRepository>();
builder.Services.AddScoped<RoleRepository>();
builder.Services.AddCors();
builder.Services.AddSignalR();
builder.Services.AddResponseCompression(opts =>
{
    opts.MimeTypes = ResponseCompressionDefaults.MimeTypes
    .Concat(new[] { "application/octet-stream" });
});
builder.Services.AddGrpc();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseWebAssemblyDebugging();
}

app.UseStaticFiles(new StaticFileOptions()
{
    FileProvider = new PhysicalFileProvider(Path.Combine(Directory.GetCurrentDirectory(),@"Images")),
    RequestPath = new Microsoft.AspNetCore.Http.PathString("/Images")
});

app.UseHttpsRedirection();
app.UseBlazorFrameworkFiles();
app.MapHub<UserHub>("/test");
app.MapControllers();
app.UseCors(o => o.AllowAnyOrigin().AllowAnyHeader().AllowAnyMethod());
app.UseGrpcWeb();
app.Run();
```

launchSettings.json
```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

ApiEndpotint для загрузки файла
```Csharp
public class UploadUserImageEndpoint : EndpointBaseAsync.WithRequest<int>.WithActionResult<string>
{
    private readonly SportStoreContext _database;
    public UploadUserImageEndpoint(SportStoreContext database)
    {
        _database = database;
    }

    [HttpPost(UploadUserImageRequest.RouteTemplate)]
    public override async Task<ActionResult<string>> HandleAsync([FromRoute] int UserId, CancellationToken cancellationToken = default)
    {
        var User = await _database.Users.SingleOrDefaultAsync(x => x.Id == UserId,cancellationToken);
        if (User is null)
        {
            return BadRequest("User does not exist.");
        }
        var file = Request.Form.Files[0];
        if (file.Length == 0)
        {
            return BadRequest("No image found.");
        }
        var filename = $"{Guid.NewGuid()}.jpg";
        var saveLocation = Path.Combine(Directory.GetCurrentDirectory(), "Images", filename);
        var resizeOptions = new ResizeOptions
        {
            Mode = ResizeMode.Pad,
            Size = new Size(640, 426)
        };
        using var image = Image.Load(file.OpenReadStream());
        image.Mutate(x => x.Resize(resizeOptions));
        await image.SaveAsJpegAsync(saveLocation,cancellationToken: cancellationToken);

        if (!string.IsNullOrWhiteSpace(User.Image))
        {
            System.IO.File.Delete(Path.Combine(Directory.GetCurrentDirectory(), "Images", User.Image));
        }

        User.Image = filename;
        await _database.SaveChangesAsync(cancellationToken);
        return Ok(User.Image);
    }
}
```




# appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",

  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "./../certificate/localhost.crt",
          "KeyPath": "./../certificate/localhost.key"
        }
      }
    }
  }
  
  
}

```

# Docker for Project.API

```
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["Example.API/Example.API.csproj", "Example.API/"]
COPY ["Example.Application/Example.Application.csproj", "Example.Application/"]
COPY ["Example.Domen/Example.Domen.csproj", "Example.Domen/"]
COPY ["Example.Infrastructure/Example.Infrastructure.csproj", "Example.Infrastructure/"]
RUN dotnet restore "./Example.API/Example.API.csproj"
COPY . .
WORKDIR "/src/Example.API"
RUN dotnet build "./Example.API.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./Example.API.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false


FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
COPY ./certificate/localhost_full.pem /app/certificate/localhost_full.pem
COPY ./certificate/localhost.key /app/certificate/localhost.key
COPY ./certificate/localhost.pfx /app/certificate/localhost.pfx
COPY ./Example.API/appsettings.json /app/appsettings.json
ENTRYPOINT ["dotnet", "Example.API.dll"]
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

# Формат даты Postgres

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



# Тестовые данные Bogus
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

- Обработка ошибок
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

# Регистрация в контейнере конкретного конструктора

```Csharp
services.AddSingleton( new EmailServerSettings ( host: "smtp.server.com", port: 25 ));
services.AddScoped( provider => new EmailServerSettings ( host: "smtp.server.com", port: 25 ));

Сервис должен использовать только те зависимости, жизненный цикл которых превышает или эквивалентен жизненному циклу сервиса. Сервис, зарегистрированный как синглтон, может безопасно использовать только singleton- зависимости. Сервис, зарегистрированный как scoped, может безопасно использовать scoped- или singleton-зависимости. Кратковременный сервис может использовать зависимости с любым жизненным циклом.
```

# Отправка токена в заголовке Authorization

```Csharp
@page "/news"


<PageTitle> Новости </PageTitle>


@using Microsoft.AspNetCore.Authorization
@using RusRoads.Domen.Models
@using System.Net.Http.Headers



@inject IHttpClientFactory factory

@if (NewsList == null)
{
    <h1>Загрузка</h1>
}
else
{
    @foreach (var n in NewsList)
    {
        
        <div class="row">
            <Widget News="@n"/>
        </div>
    }

    


}




@code {

    public IEnumerable<RusRoads.Domen.Models.News> NewsList { get; set; }

    protected override async Task OnInitializedAsync()
    {
        var token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoidXNlciIsImV4cCI6MTcxNjQwNDgyMiwiaXNzIjoiWW91cklzc3VlciIsImF1ZCI6IllvdXJBdWRpZW5jZSJ9.d5HH8AQRhKZT9yGbxX7nMT3_xfR2_-tkK3_rqoxGUt4"; // Получите этот токен от сервера аутентификации
        HttpClient http = factory.CreateClient("API");

        http.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

        NewsList = await http.GetFromJsonAsync<IEnumerable<RusRoads.Domen.Models.News>>($"{http.BaseAddress}/news");
    }

}

```
# Проверка jwt
- https://dinochiesa.github.io/jwt/


# try-catch

```Csharp
{
 // ...
}
catch (Exception ex) when (
 (ex is AggregateException
 && ex.InnerException is HttpRequestException)
 || ex is HttpRequestException
 || ex is TaskCanceledException)
{
 return null;
}
```



