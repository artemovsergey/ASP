# Identity

```
<PackageReference
 Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore"
 Version="5.0.0" />
<PackageReference
 Include="Microsoft.AspNetCore.Identity.UI" Version="5.0.0" />
```

```Csharp
services.AddDefaultIdentity<ApplicationUser>(options => options.SignIn.RequireConfirmedAccount = true).AddEntityFrameworkStores<AppDbContext>();
```

```Csharp
 app.UseRouting();
 app.UseAuthentication();
 app.UseAuthorization();
```

```Csharp
public class ApplicationUser : IdentityUser { }
```

```Csharp
public class AppDbContext : IdentityDbContext<ApplicationUser>
{
 public AppDbContext(DbContextOptions<AppDbContext> options)
 : base(options)
 { }
 public DbSet<Recipe> Recipes { get; set; }
}
```

```
dotnet ef migrations add AddIdentitySchema
```

Areas/Identity/Pages/_ViewStart.cshtml
```
@{ Layout = "/Pages/Shared/_Layout.cshtml"; }
```

_LoginPartial.cshtml
```Csharp
@using Microsoft.AspNetCore.Identity
@using RecipeApplication.Data;
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager
<ul class="navbar-nav">
@if (SignInManager.IsSignedIn(User))
{
 <li class="nav-item">
 <a class="nav-link text-dark" asp-area="Identity"
 asp-page="/Account/Manage/Index" title="Manage">Hello
 @User.Identity.Name!</a>
 </li>
 <li class="nav-item">
 <form class="form-inline" asp-page="/Account/Logout"
 asp-route-returnUrl="@Url.Page("/", new { area = "" })"
 asp-area="Identity" method="post" >
 <button class="nav-link btn btn-link text-dark"
 type="submit">Logout</button>
 </form>
 </li>
}
else
{
 <li class="nav-item">
 <a class="nav-link text-dark" asp-area="Identity"
 asp-page="/Account/Register">Register</a>
 </li>
 <li class="nav-item">
 <a class="nav-link text-dark" asp-area="Identity"
 asp-page="/Account/Login">Login</a>
 </li>
}
</ul>
```

<partial name="_LoginPartial" />

**Замечание**: есть некоторые функции, недоступные в пользовательском интерфейсе по умолчанию. Их нужно реализовать самостоятельно, например подтверждение по электронной почте и генерация QR-кода для двухфакторной аутентификации.


