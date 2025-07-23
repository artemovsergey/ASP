# ASP Core




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


































