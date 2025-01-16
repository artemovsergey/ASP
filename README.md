# ASP Core

# Project.Domen

- Models, Validations, Exceptions

```Chsarp
public abstract class Base
{
    public int Id { get; set; }
    public DateTime? CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    public DateTime? DeletedAt { get; set; }
    public bool IsDeleted { get; set; }
}
```

- Package
```xml
  <ItemGroup>
	  <PackageReference Include="FluentValidation" Version="11.9.1" />
  </ItemGroup>
```

### Валидация
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

# Project.Application
- Interfaces, Repositories, Handlers, Requests, Generics, Behaviors

## Package
- Mediatr

## ApplicationServices.cs
```Csharp
public static class ApplicationServicesRegistration
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services,IConfiguration configuration)
    {
        //services.AddLogging();

        services.AddAutoMapper(Assembly.GetExecutingAssembly());
        
        services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
        
        // Add MediatR services and register services from the current assembly
        services.AddMediatR(config => config.RegisterServicesFromAssemblies(
            Assembly.GetExecutingAssembly()));
        
        // Behaviors mediatr request pipeline
        services.AddTransient(typeof(IPipelineBehavior<,>),
            typeof(PerformanceBehavior<,>));
        services.AddTransient(typeof(IPipelineBehavior<,>),
            typeof(ValidationBehavior<,>));
        services.AddTransient(typeof(IPipelineBehavior<,>),
            typeof(UnhandledExceptionBehavior<,>));
        
        
        //services.Configure<MailSettings>(config.GetSection("MailSettings"));
        services.AddTransient<IDateTime, DateTimeService>();
        services.AddTransient<IEmailService, EmailService>();
        services.AddTransient<ICsvFileBuilder, CsvFileBuilderService>();
        
        // Redis
        
        services.AddStackExchangeRedisCache(options =>
        {
            options.Configuration = configuration.GetConnectionString("RedisConnection");
            var assemblyName = Assembly.GetExecutingAssembly().GetName();
            options.InstanceName = assemblyName.Name;
        });
        

        return services;
    }
}
```

### Request
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

### Handler
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

### Пример загрузки изображения
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

### Generic Api Result

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

### Exception

```Csharp
public class ApiException : Exception
{
    public ApiException() : base() { }
    public ApiException(string message) : base(message) { }
    
    public ApiException(string message, params object[] args)
        : base(String.Format(CultureInfo.CurrentCulture, message, args)) { }
}

public class NotFoundException : Exception
{
    public NotFoundException() : base() { }
    
    public NotFoundException(string message) : base(message) { }
    
    public NotFoundException(string message, Exception innerException) : base(message, innerException) { }
   
    public NotFoundException(string name, object key) : base($"Entity \"{name}\" ({key}) was not found.") { }
}

public class ValidationException : Exception
{
    
    public IDictionary<string, string[]> Errors { get; }
    
    public ValidationException() : base("One or more validation failures have occurred.")
    {
        Errors = new Dictionary<string, string[]>();
    }
    public ValidationException(IEnumerable<ValidationFailure> failures) : this()
    {
        var failureGroups = failures
            .GroupBy(e => e.PropertyName, e => e.ErrorMessage);
        
        foreach (var failureGroup in failureGroups)
        {  }
    }
}
```

### Behaviors

```Csharp
public class LoggingBehavior<TRequest> : IRequestPreProcessor<TRequest>
{
    private readonly ILogger<TRequest> _logger;
    
    public async Task Process(TRequest request,  CancellationToken cancellationToken)
    {
        var requestName = typeof(TRequest).Name;
        _logger.LogInformation("Request: {request}", request);
    }
}

internal class PerformanceBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly ILogger<PerformanceBehavior<TRequest, TResponse>> _logger;

    public PerformanceBehavior(ILogger<PerformanceBehavior<TRequest, TResponse>> logger)
    {
        _logger = logger;
    }
    
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        _logger.LogWarning("PerfomanceBehavior");
        
        var result = await next();
        return result;
    }
}

public class UnhandledExceptionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly ILogger<TRequest> _logger;
    
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        try { return await next(); }
        catch (Exception ex)
        {
            var requestName = typeof(TRequest).Name;
            _logger.LogError(ex, "Request: UnhandledException for Request {Name} {@Request}", requestName, request);
            throw;
        }
    }
}

public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse> where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;
    
    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators) { _validators = validators; }
    
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        if (!_validators.Any()) return await next();
        
        var context = new  ValidationContext<TRequest>(request);
        
        var validationResults = await Task.WhenAll(_validators.Select(v => v.ValidateAsync(context, cancellationToken)));
        
        var failures = validationResults.SelectMany(r => r.Errors).Where(f => f != null).ToList();
        
        return await next();
    }
}
```

### Handlers

```Chsarp
public class AddRepositoryHandler : IRequestHandler<AddRepositoryRequest, AddRepositoryRequest.Response>
{
    private readonly ProjectStoreContext _db;
    private readonly ILogger<AddRepositoryHandler> _log;
    public AddRepositoryHandler(ProjectStoreContext db, ILogger<AddRepositoryHandler> log)
    {
        _db = db;
        _log = log;
    }
    
    public async Task<AddRepositoryRequest.Response> Handle(AddRepositoryRequest request, CancellationToken cancellationToken)
    {
        _log.LogWarning("Создание объекта репозитория");

        try
        {
            _db.Repositories.Add(request.Repository);
            await _db.SaveChangesAsync();
            _log.LogInformation($"Репозиторий создан!"); 
        }
        catch (NpgsqlException e)
        {
            _log.LogError($"NpgsqlException: {e.InnerException}"); 
        }
        catch (Exception e)
        {
            _log.LogError($"Exception: {e.InnerException}");
        }
        
        //Unit.Value
        return new AddRepositoryRequest.Response(new Repository());
        
    }
}
```

```Csharp
public class RepositoryHandler : IRequestHandler<RepositoryRequest, RepositoryRequest.Response>
{
    private readonly ProjectStoreContext _db;
    private readonly ILogger<RepositoryHandler> _log;
    public RepositoryHandler(ProjectStoreContext db, ILogger<RepositoryHandler> log)
    {
        _db = db;
        _log = log;
    }

    public async Task<RepositoryRequest.Response> Handle(RepositoryRequest request, CancellationToken cancellationToken)
    {
        
        if (!string.IsNullOrEmpty(request.sortColumn) && IsValidProperty(request.sortColumn))
        {
            request = request with
            {
                sortOrder = !string.IsNullOrEmpty(request.sortOrder) && request.sortOrder.ToUpper() == "ASC"
                    ? "ASC"
                    : "DESC"
            };
        }
        
        IQueryable<Repository> newsList = _db.Repositories;

        if (!string.IsNullOrEmpty(request.filterColumn)
            && !string.IsNullOrEmpty(request.filterQuery)
            && IsValidProperty(request.filterColumn))
        {
            //users = users.Where(u => $"{filterColumn}".Contains($"{filterQuery})"));
            newsList = newsList.Where($"{request.filterColumn}.Contains(@0)", request.filterQuery);

            Console.WriteLine($"Фильтрация: {newsList.Count()}");

            // Если в базе данных нет записей, то поиск по API Github и сохранение в базу
            if (newsList.Count() == 0)
            {
                var githubpublicapi = $"https://api.github.com/search/repositories?q={request.filterQuery}";
                
                HttpClient client = new();
                client.DefaultRequestHeaders.Add("User-Agent", "YourAppName/1.0");
                HttpResponseMessage response = await client.GetAsync(githubpublicapi);
                
                string responseBody = await response.Content.ReadAsStringAsync();
                
                var jsonObject = JObject.Parse(responseBody);
                JToken items = jsonObject["items"];
                
                // Todo Настроить hash для уникальности
                foreach (JToken item in items)
                {
                    Console.WriteLine(item["name"]);
                    Console.WriteLine("--------------------------");
                    
                    var repo = new Repository
                    {
                        Name = item["name"].ToString(),
                        Owner = item["owner"]["login"].ToString(),
                        Stargazers = item["stargazers_count"].Value<int>(),
                        Watchers = item["watchers_count"].Value<int>(),
                        Url = item["html_url"].ToString()
                    };
                    _db.Repositories.Add(repo);
                    await _db.SaveChangesAsync();
                }
                
            }
            
            
        }
        
        
        var respositories = await newsList.OrderBy($"{request.sortColumn} {request.sortOrder}")
            .Skip(request.pageIndex * request.pageSize)
            .Take(request.pageSize)
            .ToListAsync();
        
        var apiresult = new ApiResult<Repository>((List<Repository>)respositories,
            _db.Repositories.Count(),
            request.pageIndex,
            request.pageSize,
            request.sortColumn,
            request.sortOrder,
            request.filterColumn,
            request.filterQuery);
        
        return new RepositoryRequest.Response(apiresult);
    }
    
    public static bool IsValidProperty(string propertyName,
        bool throwExceptionIfNotFound = true)
    {
        var prop = typeof(Repository).GetProperty(
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
}
```
```Csharp
public class DeleteRepositoryHandler : IRequestHandler<DeleteRepositoryRequest, DeleteRepositoryRequest.Response>
{
    private readonly ProjectStoreContext _db;
    private readonly ILogger<DeleteRepositoryHandler> _log;
    public DeleteRepositoryHandler(ProjectStoreContext db, ILogger<DeleteRepositoryHandler> log)
    {
        _db = db;
        _log = log;
    }
    
    public async Task<DeleteRepositoryRequest.Response> Handle(DeleteRepositoryRequest request, CancellationToken cancellationToken)
    {
        try
        {
            _db.Remove(request.Repository);
            await _db.SaveChangesAsync();
            _log.LogInformation($"Репозиторий с Id = {request.Repository.Id} удален!" );
        }
        catch (NpgsqlException e)
        {
            _log.LogError($"NpgsqlException: {e.InnerException}");
        }
        catch (Exception e)
        {
            _log.LogError($"Exception: {e.InnerException}");
        }

        return new DeleteRepositoryRequest.Response(true);
    }
}
```
```Csharp
public class EditRepositoryHandler : IRequestHandler<EditRepositoryRequest, EditRepositoryRequest.Response>
{
    private readonly ProjectStoreContext _db;
    private readonly ILogger<EditRepositoryHandler> _log;

    public EditRepositoryHandler(ProjectStoreContext db, ILogger<EditRepositoryHandler> log)
    {
        _db = db;
        _log = log;
    }


    public async Task<EditRepositoryRequest.Response> Handle(EditRepositoryRequest request,
        CancellationToken cancellationToken)
    {
        try
        {
            _db.Update(request.Repository);
            await _db.SaveChangesAsync();
            _log.LogInformation($"Репозиторий отредактирован");
        }
        catch (NpgsqlException e)
        {
            _log.LogError($"NpgsqlException: {e.InnerException}");
        }
        catch (Exception e)
        {
            _log.LogError($"Exception: {e.InnerException}");
        }

        return new EditRepositoryRequest.Response(request.Repository);
    }
}
```

### AutoMapper Profile

```Csharp
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        ApplyMappingsFromAssembly(Assembly.GetExecutingAssembly());
    }
    private void ApplyMappingsFromAssembly(Assembly
        assembly)
    {
        var types = assembly.GetExportedTypes()
            .Where(t => t.GetInterfaces().Any(i =>
                i.IsGenericType && i.GetGenericTypeDefinition()
                == typeof(IMapFrom<>)))
            .ToList();
        foreach (var type in types)
        {
            var instance = Activator.CreateInstance(type);
            var methodInfo = type.GetMethod("Mapping")
                             ??
                             type.GetInterface("IMapFrom`1").GetMethod
                                 ("Mapping");
            methodInfo?.Invoke(instance, new object[] { this
            });
        }
    }
}
```

### Requests

```Csharp
public record AddRepositoryRequest(Repository repo) : IRequest<AddRepositoryRequest.Response>
{
    public Repository Repository = repo;
    public const string RouteTemplate = "/api/Repository";
    public record Response(Repository repo);
}

public class AddRepositoryRequestValidator :  AbstractValidator<AddRepositoryRequest>
{
    private readonly ProjectStoreContext _context;
    public AddRepositoryRequestValidator(ProjectStoreContext context)
    {
        _context = context;
        RuleFor(v => v.Repository.Name)
            .NotEmpty().WithMessage("Name is required");
    }
}

public class DeleteRepositoryRequest(Repository repo) : IRequest<DeleteRepositoryRequest.Response>
{
    public Repository Repository = repo;
    public record Response(bool Result);
}

public class EditRepositoryRequest(Repository repo) : IRequest<EditRepositoryRequest.Response>
{
    public string RouteTemplate = "/api/Repository";

    public Repository Repository = repo;
    public record Response(Repository repo);
}

public record RepositoryRequest(string? sortColumn,
                                string? sortOrder,
                                int pageIndex,
                                int pageSize,
                                string? filterColumn,
                                string? filterQuery) : IRequest<RepositoryRequest.Response>
{
    public record Response(ApiResult<Repository> result);
}

```

# Project.Infrastructure
- Configurations, Migrations, Data/Context.cs

## Package
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
      <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.0.4" />
    </ItemGroup>
```

## InfrastructureServices
```Csharp
public static class InfrastructureServices
{
    public static IServiceCollection AddInfrastructureServices(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddScoped<JwtHandler>();
        services.AddDbContext<ProjectStoreContext>(opts =>
        {
            opts.UseNpgsql(configuration.GetConnectionString("PostgreSQL"));
        }).AddLogging();


        if (configuration["ASPNETCORE_ENVIRONMENT"] == "Production")
        {
            services.AddDbContext<ApplicationContext>(options =>
                options.UseSqlServer(
                    configuration.GetConnectionString("MSSQL")
                )
            );
        }
        else
        {
            services.AddDbContext<ApplicationContext>(options =>
                options.UseSqlServer(
                    configuration.GetConnectionString("DefaultConnection")
                )
            );
        }
        
        return services;
    }
}
```


## EF Configuration
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

## Context
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

    public SportStoreContext(DbContextOptions<ColledgeContext> opt) : base(opt)
    {
        //Database.EnsureDeleted();
        Database.EnsureCreated();
        //Database.Migrate();
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

## Factory
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


# Project.API

ApiEndPoints, Controllers, Hubs, Images, Services, Filters, Graphql, Helpers, Hubs, Logs

## Package
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

```
   <ItemGroup>
        <PackageReference Include="Asp.Versioning.Mvc" Version="8.1.0" />
        <PackageReference Include="Asp.Versioning.Mvc.ApiExplorer" Version="8.1.0" />
        <PackageReference Include="CsvHelper" Version="33.0.1" />
        <PackageReference Include="FluentResults" Version="3.16.0" />
        <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
        <PackageReference Include="HotChocolate.AspNetCore" Version="13.9.9" />
        <PackageReference Include="HotChocolate.AspNetCore.Authorization" Version="13.9.9" />
        <PackageReference Include="HotChocolate.Data.EntityFramework" Version="13.9.9" />
        <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.8" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="8.0.0" />
        <PackageReference Include="Serilog" Version="4.0.1" />
        <PackageReference Include="Serilog.AspNetCore" Version="8.0.2" />
        <PackageReference Include="Serilog.Exceptions.EntityFrameworkCore" Version="8.4.0" />
        <PackageReference Include="Serilog.Settings.Configuration" Version="8.0.2" />
        <PackageReference Include="Serilog.Sinks.Console" Version="6.0.0" />
        <PackageReference Include="Serilog.Sinks.File" Version="6.0.0" />
        <PackageReference Include="Swashbuckle.AspNetCore" Version="6.4.0"/>
        <PackageReference Include="Swashbuckle.AspNetCore.Annotations" Version="6.7.0" />
    </ItemGroup>
```

## ApiExtensions

```Csharp
public static class SwaggerServices
{
    public static void UseSwaggerServices(this IApplicationBuilder app)
    {
        app.UseSwagger();
        app.UseSwaggerUI(
            options =>
            {
                foreach (var description in new List<string>(){"v1","v2"})
                {
                    options.SwaggerEndpoint($"/swagger/{description}/swagger.json", description.ToUpperInvariant());
                }
            });
    }
}
```

## BaseController
```Csharp
[ApiVersion("1.0", Deprecated = true)]
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]

public class BaseController : ControllerBase
{
    private IMediator _mediator;
    protected IMediator Mediator => _mediator ??=  HttpContext.RequestServices.GetService<IMediator>();  
}
```

## Controller v1
```Csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class RepositoryController : ControllerBase
{
    private readonly IMediator _mediator;
    private readonly ILogger<RepositoryController> _logger;
    public RepositoryController(IMediator mediatr, ILogger<RepositoryController> logger)
    {
        _logger = logger;
        _mediator = mediatr;
    }
    
    //[Authorize]
    //[Authorize(Roles = "Administrator")]
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [SwaggerOperation(Summary = "Получает все репозитории.")]
    [SwaggerResponse(StatusCodes.Status200OK, "All repo successfully retrieved")]
    [SwaggerResponse(StatusCodes.Status404NotFound, "Repo not found", typeof(ValidationProblemDetails))]
    public async Task<IActionResult> GetAllRepositories(string? sortColumn = null,
        string? sortOrder = null,
        int pageIndex = 0,
        int pageSize = 10,
        string? filterColumn = null,
        string? filterQuery = null)
    {
        var response = await _mediator.Send(new RepositoryRequest(sortColumn,sortOrder,pageIndex,pageSize,filterColumn,filterQuery));
        
        if(response.result == null)
        {
            return NotFound("Репозитории не найдены");
        }
        return Ok(response.result);
    }

   
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [SwaggerOperation(Summary = "Создание репозитория.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Repo created success")]
    [SwaggerResponse(StatusCodes.Status400BadRequest, "Repo not created", typeof(ValidationProblemDetails))]
    public async Task<ActionResult<Repository>> PostRepository([FromBody] Repository repo)
    {
        var response = await _mediator.Send(new AddRepositoryRequest(repo));
        if(response.repo == null)
        {
            return BadRequest("Репозиторий не создан!");
        }
        return Ok("Новый репозиторий создан!");
    }
    
    [Authorize]
    [HttpPut]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [SwaggerOperation(Summary = "Редактирование репозитория.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Repo edited success")]
    [SwaggerResponse(StatusCodes.Status400BadRequest, "Repo not edited", typeof(ValidationProblemDetails))]
    public async Task<ActionResult<Repository>> EditRepository([FromBody] Repository repo)
    {
        var response = await _mediator.Send(new EditRepositoryRequest(repo));
        if(response.repo == null)
        {
            return BadRequest("Репозиторий не отредактирован!");
        }
        return Ok("Репозиторий отредактирован!");
    }
    
    [Authorize]
    [HttpDelete]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [SwaggerOperation(Summary = "Удаление репозитория.")]
    [SwaggerResponse(StatusCodes.Status200OK, "Repo deleted success")]
    [SwaggerResponse(StatusCodes.Status400BadRequest, "Repo not deleted", typeof(ValidationProblemDetails))]
    public async Task<ActionResult<bool>> DeleteRepository([FromBody] Repository repo)
    {
        var response = await _mediator.Send(new DeleteRepositoryRequest(repo));
        if(response.Result == false)
        {
            return BadRequest("Репозиторий не удален!");
        }
        return Ok("Репозиторий удален!");
    }
}
```

## TokenController
```Csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class TokenController : ControllerBase
{
    private readonly IConfiguration _config;

    public TokenController(IConfiguration config)
    {
        _config = config;
    }

    [HttpGet]
    public IActionResult GenerateToken()
    {
        var claims = new List<Claim> { new Claim(ClaimTypes.Name, "user") };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:SecretKey"]));
        Console.WriteLine(key);

        // создаем JWT-токен
        var jwt = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.Add(TimeSpan.FromDays(365)),
            signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256));
        
        return Ok(new JwtSecurityTokenHandler().WriteToken(jwt));
    }
}
```

## Controller with Redis
```Csharp
[ApiVersion("1.0")]
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IMemoryCache _memoryCache;
    private readonly ApplicationContext _db;
    private readonly IDistributedCache _distributedCache;

    public UsersController(IUserService userService,
                           IMemoryCache memoryCache,
                           ApplicationContext db,
                           IDistributedCache distributedCache)
    {
        _userService = userService;
        _memoryCache = memoryCache;
        _db = db;
        _distributedCache = distributedCache;
    }

    [HttpPost]
    public IActionResult PostUser([FromBody] ApplicationUser user)
    {
        try
        {
            _db.Users.Add(user);
            _db.SaveChanges();
            return Ok(user);
        }
        catch (Exception e)
        {
            return BadRequest($"Bad Request: {e.InnerException.Message}");
            throw;
           
        }
    }

    [AspCore.Authorize]
    [HttpPost]
    [Route("login")]
    public async Task<ActionResult> LoginUser(ApplicationUser user)
    {
        var result = await _db.Users.Where(u => u.Email == user.Email).FirstOrDefaultAsync();
        if (result != null)
        {
            return Ok(result.Id);
        }
        else
        {
            return NotFound("Неправильный email или password");
        }
    }

    [HttpPost("auth")]
    public IActionResult Authenticate([FromBody] AuthenticateRequest model)
    {
        Console.WriteLine("Метод контроллера Authenticate");
        var response = _userService.Authenticate(model);
        
        if (response == null)
            return BadRequest(new { message = "Username or password is incorrect" });
        return Ok(response);
    }

    /*
     Кэширование в памяти не рекомендуется использовать 
     при работе нескольких экземпляров одного решения,
     поскольку данные не будут согласованы.
     */
    
    [HttpGet("{cakeName}")]
    public async Task<ApplicationUser> Get(string cakeName)
    {
        var cacheKey = cakeName.ToLower();
        if (!_memoryCache.TryGetValue(cacheKey, out ApplicationUser cakeList))
        {
            cakeList = _userService.GetById(1);
            var cacheExpirationOptions = new MemoryCacheEntryOptions
            {
                AbsoluteExpiration = DateTime.Now.AddHours(6),
                Priority = CacheItemPriority.Normal, SlidingExpiration = TimeSpan.FromMinutes(5)
            };
            _memoryCache.Set(cacheKey, cakeList, cacheExpirationOptions);
        }

        return cakeList;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<ApplicationUser>>> GetUsers(CancellationToken cancellationToken)
    {
        const string cacheKey = "GetUsers";
        IEnumerable<ApplicationUser> users;
        var cacheUsers = await _distributedCache.GetAsync(cacheKey, cancellationToken);

        if (cacheUsers == null)
        {
            users = _db.Users.ToList();
            var serializedUsers = JsonConvert.SerializeObject(users);
            cacheUsers = Encoding.UTF8.GetBytes(serializedUsers);

            var options = new DistributedCacheEntryOptions()
                .SetAbsoluteExpiration(DateTime.Now.AddMinutes(5))
                .SetSlidingExpiration(TimeSpan.FromMinutes(1));
            
            await _distributedCache.SetAsync(cacheKey,cacheUsers, options, cancellationToken);
            return Ok(users);
        }
        
        var result  = Encoding.UTF8.GetString(cacheUsers);
        users = JsonConvert.DeserializeObject<IEnumerable<ApplicationUser>>(result);
        return Ok(users);
    }
}
```

## AccountController
```Csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]/[action]")]
public class AccountController : ControllerBase
{
    private readonly ApplicationContext _context;
    private readonly JwtHandler _jwtHandler;
    
    public AccountController(
        ApplicationContext context,
        JwtHandler jwtHandler)
    {
        _context = context;
        _jwtHandler = jwtHandler;
    }
    
    [HttpPost]
    public async Task<IActionResult> Login([FromBody] LoginRequest loginRequest)
    {
        var user = await _context.Users.Where(u => u.Email == loginRequest.Email).FirstOrDefaultAsync();
        var password = await _context.Users.Where(u => u.Password == loginRequest.Password).FirstOrDefaultAsync();
        
        if (user == null
            || password == null)
            return Unauthorized(new LoginResult() {
                Success = false,
                Message = "Invalid Email or Password."
            });
        var secToken = await _jwtHandler.GetTokenAsync(user);
        var jwt = new JwtSecurityTokenHandler().WriteToken(secToken);
        return Ok(new LoginResult() {
            Success = true, Message = "Login successful", Token = jwt
        });
    }
    
}
```

## Program
```Csharp
var builder = WebApplication.CreateBuilder(args);

// Log.Logger = new LoggerConfiguration().ReadFrom.Configuration(builder.Configuration).CreateLogger();
// builder.Host.UseSerilog();

var configuration = builder.Configuration;

var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(configuration["Jwt:SecretKey"]));


Console.WriteLine(key);

builder.Services.AddApplicationServices(builder.Configuration);
builder.Services.AddInfrastructureServices(builder.Configuration);

builder.Services.AddInfrastructureIdentity(builder.Configuration);


builder.Services.AddHttpContextAccessor();
builder.Services.AddMemoryCache();

//builder.Services.AddTransient<IConfigureOptions<SwaggerGenOptions>,ConfigureSwaggerOptions>();

builder.Services.AddApiVersioning(config =>
{
    config.DefaultApiVersion = new ApiVersion(1, 0);
    config.AssumeDefaultVersionWhenUnspecified = true;
    config.ReportApiVersions = true;
})
    .AddApiExplorer(
    options =>
    {
        options.GroupNameFormat = "'v'VVV";
        options.SubstituteApiVersionInUrl = true;
    });;


builder.Services.AddSwaggerGen(c =>
{
    //c.OperationFilter<SwaggerDefaultValues>();
    c.EnableAnnotations();
    
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Репозитории", Version = "v2024" });
    c.SwaggerDoc("v2", new OpenApiInfo { Title = "Репозитории", Version = "v2024" });
    
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "Authorization using jwt token. Example: \"Bearer {token}\"",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "bearer"
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

builder.Services.AddCors();

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = configuration["Jwt:Issuer"],
            ValidAudience = configuration["Jwt:Audience"],
            IssuerSigningKey = key,
            //IssuerSigningKey = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes(builder.Configuration["JwtSettings:SecurityKey"]))
        };
    });


builder.Services.AddGraphQLServer()
    .AddAuthorization()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddFiltering()
    .AddSorting();


builder.Services.AddSignalR();

builder.Services.AddAuthorization();

// builder.Services.AddControllers(options =>
//     options.Filters.Add(new ApiExceptionFilter()));

// builder.Services.Configure<ApiBehaviorOptions>(options =>
//     options.SuppressModelStateInvalidFilter = true
// );

builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();


//app.UseSerilogRequestLogging();

if (app.Environment.IsDevelopment())
{
    Console.WriteLine("Env: Development");
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
     app.UseSwaggerUI(
         options =>
         {
             var descriptions = app.DescribeApiVersions();
             foreach (var description in descriptions)
             {
                 options.SwaggerEndpoint($"/swagger/{description.GroupName}/swagger.json", description.GroupName.ToUpperInvariant());
                 
             }
         });
}
else
{
    Console.WriteLine("Env: Production");
    // For test in Production 
    app.UseSwaggerServices();
    
    app.UseExceptionHandler("/Error");
    app.MapGet("/Error", () => Results.Problem());
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseCors(o => o.AllowAnyOrigin().AllowAnyHeader().AllowAnyMethod());

// Identity Project
//app.UseMiddleware<JwtMiddleware>();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.MapGraphQL("/api/graphql");

app.MapHub<HealthCheckHub>("/api/health-hub");

app.Run();
```

## launchSettings.json
```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

## ApiEndpotint для загрузки файла
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

## appsettings.Development.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",

  "ConnectionStrings": {
    "MSSQL": "Server=localhost,1433;Database=Colledge;Trust Server Certificate=True;MultipleActiveResultSets=true",
    "MSSQLAuth": "Server=localhost,1433;Database=Colledge;UserId=yourUsername;Password=yourPassword;MultipleActiveResultSets=true;",
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=Colledge;Trusted_Connection=True;MultipleActiveResultSets=true",
    "PostgreSQL": "Host=localhost;Port=5432;Database=Colledge;Username=postgres;Password=root",
    "SQLite" : "Data Source = TodoApp.db;Foreign Keys=True;"
  },

  "Jwt": {
    "SecretKey": "asdasdsdsfdgdgfdgdgdgdgdgdsdasdsadgeregegerggdfgdga",
    "Issuer": "YourIssuer",
    "Audience": "YourAudience",
    "ExpirationTimeInMinutes": 30
  },

  "DefaultPasswords": {
    "RegisteredUser": "Sampl3Pa$$_User",
    "Administrator": "Sampl3Pa$$_Admin"
  },


  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5002"
      },
      "Https": {
        "Url": "https://localhost:5001",
        //"Certificate": {
        //  "Path": "./../certificate/localhost.crt",
        //  "KeyPath": "./../certificate/localhost.key"
        //}
      }
    }
  }
}
```

## appsettings.json
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",

  "ConnectionStrings": {
    "MSSQL": "Server=mssql;Database=ProjectStore;Trust Server Certificate=True;User Id=sa;Password=HlxTm2fcFE54JA1I_Yp5;MultipleActiveResultSets=true",
    "MSSQLAuth": "Server=localhost,1433;Database=ProjectStore;UserId=yourUsername;Password=yourPassword;MultipleActiveResultSets=true;",
    "DefaultConnection": "Server=mssql;Database=ProjectStore;Trusted_Connection=True;MultipleActiveResultSets=true",
    "PostgreSQL": "Host=db;Port=5432;Database=ProjectStore;Username=postgres;Password=root"
  },

  "Jwt": {
    "SecretKey": "asdasdsdsfdgdgfdgdgdgdgdgdsdasdsadgeregegerggdfgdga",
    "Issuer": "YourIssuer",
    "Audience": "YourAudience",
    "ExpirationTimeInMinutes": 30
  },

  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://api:5002"
      },
      "Https": {
        "Url": "https://api:5001",
        //"Certificate": {
        //  "Path": "/app/certificate/localhost.crt",
        //  "KeyPath": "/app/certificate/localhost.key"
        //}
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

# ExceptionHandlingMiddlwere
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

```Csharp
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration).CreateLogger();
builder.Host.UseSerilog();
```

appsettings.json
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

# Tuple

tuple - можно применять как аналог DTO или ViewModel при отправке ответа

# String

https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/

# Learn

- using static
- reflection
- lambda
- null 
- оператор _
- record
- generic
- Action
- Func

# Rest Controller

```Csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoApp.API.Data;
using TodoApp.API.Models;

namespace TodoApp.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class TasksController(TodoAppContext db) : ControllerBase
{

    [HttpGet]
    [Route("/api/Category/{categoryId}/tasks")]
    public async Task<ActionResult<IEnumerable<TaskEntity>>> GetTasksByCategory(int categoryId){
        
        IQueryable<TaskEntity> tasks;

        if(categoryId != 0){
           tasks = db.Tasks.Include(t => t.Category).Include(t => t.Priority).Where(t => t.CategoryId == categoryId);
        }
        else{
           return await db.Tasks.Include(t => t.Category).Include(t => t.Priority).ToListAsync();
        }
        
        return await tasks.ToListAsync();
    }

    /// <summary>
    /// Вывод списка задач
    /// </summary>
    /// <returns> Коллекцию задач </returns>
    /// 
    [HttpGet]
    public async Task<ActionResult<IEnumerable<TaskEntity>>> GetTasks(){
        return Ok(await db.Tasks.Include(t => t.Category).Include(t => t.Priority).ToListAsync());
    }

    /// <summary>
    /// Создание задачи
    /// </summary>
    /// <param name="t"> объект задачи TaskEntity</param>
    /// <returns> Код 201 и объект TaskEntity</returns>
    [HttpPost]
    public async Task<ActionResult<TaskEntity>> CreateTask(TaskEntity t){
        
        try
        {
           db.Tasks.Add(t);
           await db.SaveChangesAsync();
        }
        catch (Exception ex)
        {
           throw new Exception($"{ex.InnerException?.Message}");
        } 

        return Created("",t);
    }

    /// <summary>
    /// Обновлене задачи
    /// </summary>
    /// <param name="t"></param>
    /// <returns></returns>
    
    [HttpPut]
    public async Task<ActionResult<TaskEntity>> PutTask(TaskEntity t){
        if(!await TaskExist(t.Id)) return NotFound($"Нет задачи с id = {t.Id}"); 
        db.Update(t);
        await db.SaveChangesAsync();
        return Ok(t);
    }

    /// <summary>
    /// Удаление задачи
    /// </summary>
    /// <param name="t"></param>
    /// <returns></returns>
    [HttpDelete("{id}")]
    public async Task<ActionResult<bool>> DeleteTask([FromRoute] int id){

        if(!await TaskExist(id)) return NotFound($"Нет задачи с id = {id}");
        var task = await db.Tasks.FindAsync(id);
        db.Tasks.Remove(task!);
        await db.SaveChangesAsync();

        return true; 
    }

    private async Task<bool> TaskExist(int taskId){
      return await db.Tasks.AnyAsync<TaskEntity>(e => e.Id == taskId) ? true : false;
    }
    
}
```
