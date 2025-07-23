# ASP Core



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


































