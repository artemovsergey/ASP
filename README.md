# Program

# LoggingServices



# SwaggerServices

```csharp
public static class SwaggerServices
{
    public static IServiceCollection AddSwaggerServices(
        this IServiceCollection services
    )
    {
        services.AddOpenApi();
        
        services.AddSwaggerGen(c =>
        {
            c.SwaggerDoc(
                "v1",
                new OpenApiInfo
                {
                    Title = "Sample App API",
                    Version = "v1",
                    Description = "API для управления пользователями",
                    Contact = new OpenApiContact
                    {
                        Name = "Your Name",
                        Email = "your.email@example.com",
                    },
                }
            );

            c.SwaggerDoc(
                "v2",
                new OpenApiInfo
                {
                    Title = "Minimal API 2",
                    Version = "v2",
                    Description = "API 2 для управления пользователями",
                    Contact = new OpenApiContact
                    {
                        Name = "Your Name",
                        Email = "your.email@example.com",
                    },
                }
            );


            c.EnableAnnotations();
            c.AddSecurityDefinition(
                "Bearer",
                new OpenApiSecurityScheme
                {
                    Description = "Authorization using jwt token. Example: \"Bearer {token}\"",
                    Name = "Authorization",
                    In = ParameterLocation.Header,
                    Type = SecuritySchemeType.ApiKey,
                }
            );

            c.AddSecurityRequirement(
                new OpenApiSecurityRequirement
                {
                    {
                        new OpenApiSecurityScheme
                        {
                            Reference = new OpenApiReference
                            {
                                Type = ReferenceType.SecurityScheme,
                                Id = "Bearer",
                            },
                        },
                        new string[] { }
                    },
                }
            );
        });
        
        services.AddEndpointsApiExplorer();

        return services;
    }
}
```

# ApiVersioningServices

```csharp
public static class ApiVersioningServices
{
    public static IServiceCollection AddApiVersioningServices(this IServiceCollection services)
    {
        services.AddApiVersioning(options =>
        {
            options.DefaultApiVersion = new ApiVersion(1, 0);
            options.AssumeDefaultVersionWhenUnspecified = true;
            options.ReportApiVersions = true;
            options.ApiVersionReader = new UrlSegmentApiVersionReader();
        }).AddApiExplorer(options =>
        {
            options.GroupNameFormat = "'v'VVV";
            options.SubstituteApiVersionInUrl = true;
        });
        
        return services;
    }
}
```

# JwtServices

```csharp
public static class JwtServices
{
    public static IServiceCollection AddJwtServices(
        this IServiceCollection services,
        IConfiguration configuration
    )
    {
        // TODO Включить обязательные проверки JWT
        services
            .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.SaveToken = true;
                options.RequireHttpsMetadata = false;
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = false,
                    ValidateAudience = false,
                    ValidateLifetime = false,
                    ValidateIssuerSigningKey = true,

                    IssuerSigningKey = new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes(configuration["TokenPublicKey"]!)
                    ),
                };
            });
        
        services.AddAuthorization();
       
        return services;
    }
}
```

# MediatrServices

```csharp
builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly);
});
```


# HealthServices

```csharp
public static class HealthServices
{
    public static IServiceCollection AddHealthServices(
        this IServiceCollection services
    )
    {
        services.AddHealthChecks().AddCheck<DatabaseHealthCheck>("Database");
        
        return services;
    }
}

public class DatabaseHealthCheck(SampleAppContext db) : IHealthCheck
{
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = new CancellationToken())
    {
        var connect = await db.Database.CanConnectAsync();
        if (connect) return HealthCheckResult.Healthy();
        else return HealthCheckResult.Unhealthy();
    }
}
```

# ExceptionHandlerMiddleware

```csharp
public class ExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlerMiddleware> _logger;
    private readonly IHostEnvironment _env;

    public ExceptionHandlerMiddleware(
        RequestDelegate next,
        ILogger<ExceptionHandlerMiddleware> logger,
        IHostEnvironment env)
    {
        _next = next;
        _logger = logger;
        _env = env;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception has occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/problem+json"; // Стандартный ContentType для ProblemDetails
        
        var problemDetails = new ProblemDetails
        {
            Instance = context.Request.Path,
            Detail = _env.IsDevelopment() ? exception.StackTrace : null,
            Extensions = { ["traceId"] = context.TraceIdentifier }
        };

        switch (exception)
        {
            case Npgsql.PostgresException ex:
                problemDetails.Title = "Database error";
                problemDetails.Status = (int)HttpStatusCode.InternalServerError;
                problemDetails.Detail = ex.Message;
                problemDetails.Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.1";
                break;
                
            case BadHttpRequestException ex:
                problemDetails.Title = "Invalid request";
                problemDetails.Status = (int)HttpStatusCode.BadRequest;
                problemDetails.Detail = ex.Message;
                problemDetails.Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.1";
                break;
                
            case NotFoundException ex:
                problemDetails.Title = "Not found";
                problemDetails.Status = (int)HttpStatusCode.NotFound;
                problemDetails.Detail = ex.Message;
                problemDetails.Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.4";
                break;
                
            case SqlTypeException ex:
                problemDetails.Title = "Data type error";
                problemDetails.Status = (int)HttpStatusCode.BadRequest;
                problemDetails.Detail = ex.Message;
                problemDetails.Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.1";
                break;
                
            default:
                problemDetails.Title = "An unexpected error occurred";
                problemDetails.Status = (int)HttpStatusCode.InternalServerError;
                problemDetails.Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.1";
                break;
        }

        context.Response.StatusCode = problemDetails.Status.Value;
        
        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            WriteIndented = _env.IsDevelopment()
        };
        
        return context.Response.WriteAsync(JsonSerializer.Serialize(problemDetails, options));
    }
}
```

# SecurityMiddleware

```csharp
public static class SecurityMiddlewares
{
    public static WebApplication UseSecurityMiddlewares(
        this WebApplication app
    )
    {
        if (app.Environment.IsProduction())
        {
            app.UseHsts();
            app.UseHttpsRedirection();
        }

        app.UseCors(option => option.WithOrigins("https://localhost:4200").AllowAnyHeader().AllowAnyMethod());

        app.UseRateLimiter();
        
        return app;
    }
}
```

# EndpointsMiddleware

```csharp
public static class UsersEndpoints
{
    public static WebApplication UseUsersEndpoints(this WebApplication app)
    {
        var apiVersionSet = app.NewApiVersionSet()
            .HasApiVersion(new ApiVersion(1, 0))
            .HasApiVersion(new ApiVersion(2, 0))
            .ReportApiVersions()
            .Build();

        app.MapGet(
            "api/users",
            async (IMediator mediatr, ILogger<Program> log, [AsParameters] UserRequestOption opt) =>
            {
                var request = new GetUsersRequest(opt);
                var response = await mediatr.Send(request);

                log.LogInformation("Вывод всех пользователей");
                return Results.Ok(response.Users);
            }
        ).WithOpenApi();

        app.MapPost("api/users/login", async (UserDto userDto, IUserRepository repo) =>
            {
                // var userMapping = userDto.MapToUser();
                var user = await repo.FindUserByLoginAsync(userDto.Login);
                return CheckPasswordHash(userDto, user);
            })
            .WithOpenApi();

        
        // TODO Генерация JWT токена при создании пользователя (обычно генерируется при логине)
        // TODO Хранение токена в БД (нестандартная практика)
        if (app.Environment.IsDevelopment())
        {
            app.MapGet(
                "/generate",
                async (SampleAppContext db, ITokenService tokenService, CancellationToken ct) =>
                {
                    using var hmac = new HMACSHA512();

                    Faker<UserDto> faker = new Faker<UserDto>("en")
                        .RuleFor(u => u.Login, f => GenerateLogin(f).Trim())
                        .RuleFor(u => u.Password, f => GeneratePassword(f).Trim().Replace(" ", ""));

                    string GenerateLogin(Faker faker)
                    {
                        return faker.Random.Word() + faker.Random.Number(3, 5);
                    }

                    string GeneratePassword(Faker faker)
                    {
                        return faker.Random.Word() + faker.Random.Number(3, 5);
                    }

                    var users = faker.Generate(100).Where(u => u.Login.Length > 4 && u.Login.Length <= 10);

                    List<User> userToDb = new List<User>();

                    try
                    {
                        foreach (var user in users)
                        {
                            var u = new User()
                            {
                                Login = user.Login,
                                PasswordHash = hmac.ComputeHash(Encoding.UTF8.GetBytes(user.Password)),
                                PasswordSalt = hmac.Key,
                                Token = tokenService.CreateToken(user.Login),
                            };
                            userToDb.Add(u);
                        }

                        await db.Users.AddRangeAsync(userToDb, ct);
                        await db.SaveChangesAsync(ct);
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"{ex.InnerException?.Message}");
                    }

                    return Results.Ok($"Данные сгенерированы: \n {userToDb.Count}");
                }
            ); 
        }

        app.MapGet("token",
            (IConfiguration config) =>
            {
                var claims = new List<Claim> { new Claim(ClaimTypes.Name, "user") };
                var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["TokenPublicKey"]!));

                // создаем JWT-токен
                var jwt = new JwtSecurityToken(
                    // issuer: _config["Jwt:Issuer"],
                    // audience: _config["Jwt:Audience"],
                    claims: claims,
                    expires: DateTime.UtcNow.Add(TimeSpan.FromDays(365)),
                    signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
                );

                return Results.Ok(new JwtSecurityTokenHandler().WriteToken(jwt));
            }
        );

        app.MapGet("api/getToken2", (IConfiguration config) =>
        {
            var claims = new List<Claim> { new Claim(ClaimTypes.Name, "user") };

            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["Jwt:SecretKey"]!));
            Console.WriteLine(key);

            // создаем JWT-токен
            var jwt = new JwtSecurityToken(
                issuer: config["Jwt:Issuer"],
                audience: config["Jwt:Audience"],
                claims: claims,
                expires: DateTime.UtcNow.Add(TimeSpan.FromDays(365)),
                signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256));

            return Results.Ok(new JwtSecurityTokenHandler().WriteToken(jwt));
        });


        return app;
    }

    private static IResult CheckPasswordHash(UserDto userDto, User user)
    {
        using var hmac = new HMACSHA512(user.PasswordSalt!);
        var computedHash = hmac.ComputeHash(Encoding.UTF8.GetBytes(userDto.Password));

        for (int i = 0; i < computedHash.Length; i++)
        {
            if (user.PasswordHash != null && computedHash[i] != user.PasswordHash[i])
            {
                return Results.Unauthorized();
            }
        }

        return Results.Ok(user);
    }
}
```

# SwaggerMiddleware

```csharp
public static class SwaggerMiddlewares
{
    public static WebApplication UseSwaggerMiddlewares(
        this WebApplication app
    )
    {
        app.UseSwagger();
        app.UseSwaggerUI(c =>
        {
            c.SwaggerEndpoint("/swagger/v1/swagger.json", "Sample App API V1");
            c.SwaggerEndpoint("/swagger/v2/swagger.json", "Sample App API V2");
            c.RoutePrefix = string.Empty;
        });
        app.MapOpenApi();

        return app;
    }
}
```

# HealthMIddleware

```csharp
public static class HealthMiddlewares
{
    public static WebApplication UseHealthMiddlewares(
        this WebApplication app
    )
    {
        app.MapHealthChecks("/health", new HealthCheckOptions
        {
            ResponseWriter = async (context, report) =>
            {
                var result = new
                {
                    Status = report.Status.ToString(),
                    Checks = report.Entries.Select(e => new
                    {
                        Component = e.Key,
                        Status = e.Value.Status.ToString(),
                        Description = e.Value.Description,
                        Exception = e.Value.Exception?.Message
                    })
                };

                context.Response.ContentType = "application/json";
                await context.Response.WriteAsync(JsonSerializer.Serialize(result));
            }
        }).AddEndpointFilter((context, next) =>
        {
            var filter = new ApiExceptionEndpointFilter();
            return filter.InvokeAsync(context, next);
        });

        return app;
    }
}
```



















