# Application

```cproj
<ItemGroup>
    <PackageReference Include="FluentResults" Version="3.15.2" />
    <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
    <PackageReference Include="Mapster" Version="7.4.0" />
    <PackageReference Include="MediatR" Version="12.1.1" />
    <PackageReference Include="Microsoft.IdentityModel.Tokens" Version="7.0.3" />
    <PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="7.0.3" />
</ItemGroup>
```

```FluentResults``` - Это библиотека для .NET, которая позволяет возвращать результаты выполнения методов в более читаемом и удобном формате. Она предоставляет класс Result, который может быть использован для обозначения успешного или неуспешного выполнения операции и включает информацию об ошибках.

```FluentValidation.AspNetCore``` - Это пакет FluentValidation, специально предназначенный для ASP.NET Core. Он используется для создания валидаторов с использованием строго типизированных объектов, что делает код более понятным и безопасным.

```Mapster``` - это быстрый, легкий и мощный маппер для .NET. Он позволяет копировать данные между объектами различных типов, поддерживает трансформацию и преобразование данных.

```MediatR``` - это библиотека для .NET, которая реализует шаблон проектирования Mediator для упрощения взаимодействия между компонентами системы. Она позволяет изолировать вызывающий код от вызываемого кода, что улучшает тестируемость и модульность приложения.

```Microsoft.IdentityModel.Tokens``` - Это библиотека, которая предоставляет набор классов для работы с токенами безопасности. Она включает в себя классы для создания и проверки JSON Web Tokens (JWT).

```System.IdentityModel.Tokens.Jwt``` - Это часть Microsoft.IdentityModel.Tokens, которая предоставляет классы для работы с JSON Web Tokens (JWT). Эти классы могут быть использованы для создания, декодирования и проверки JWT.

# Сервисы для Application

ApplicationServicesRegistration.cs

```Csharp
// Importing the required libraries
using Application.Features.Users.Create;
using Application.Pipelines;
using Microsoft.Extensions.DependencyInjection;
using System.Reflection;

// Namespace for the application
namespace Application
{
    // Static class for registering application services
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
}

```

# Infrastructure

```Csharp
using Application.Shared;
using Domain.IServices;
using Domain.Repositories;
using Infrastructure.DataBase;
using Infrastructure.Repositories;
using Infrastructure.Services;
using Infrastructure.Services.TelegramService;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace Infrastructure
{
    // Static class for registering infrastructure services
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
}

```

# Presentation

Program.cs
```Csharp
using Application;
using Infrastructure;
using Microsoft.AspNetCore.Authorization;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using Serilog;
using Swashbuckle.AspNetCore.Filters;
using System.Text;
using WebApi.Middleware;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog for logging
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration).CreateLogger();
builder.Host.UseSerilog();

// Add controllers and API explorer
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

// Configure CORS to allow any method, origin, and header
builder.Services.AddCors(opts => opts.AddDefaultPolicy(opts => opts.AllowAnyMethod().AllowAnyOrigin().AllowAnyHeader()));

builder.Services.AddSwaggerExamplesFromAssemblyOf<Program>();

// Configure Swagger for API documentation
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

// Configure authentication with JWT
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

// Configure authorization to require authenticated user
builder.Services.AddAuthorization(opts =>
{
    opts.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Add application and infrastructure services
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);

// Add middleware for exception handling
builder.Services.AddTransient<ExceptionHandlingMiddlwere>();

var app = builder.Build();

// Use Swagger in development environment
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

// Configure middleware, CORS, authentication, and authorization
app.UseMiddleware<ExceptionHandlingMiddlwere>();
app.UseHttpsRedirection();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();

// Map controllers and run application
app.MapControllers();
app.Run();

```













