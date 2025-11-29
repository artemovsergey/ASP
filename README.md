
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



















