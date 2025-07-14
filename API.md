# Настройки сериализации

# Enums

```Csharp
builder.Services.Configure<JsonOptions>(options =>
{
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
});
```

# Newtonsoft

```Csharp

// если происходит цикл при связи 1:M через API
builder.Services.AddControllersWithViews()
    .AddNewtonsoftJson(options =>
    options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
);
```

# Option json

```Csharp
builder.Services.AddControllers()
 .AddJsonOptions(options =>
 {
 options.JsonSerializerOptions.WriteIndented = true;
 });
```













