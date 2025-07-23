# ASP Core




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


































