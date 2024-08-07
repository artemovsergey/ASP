# GraphQL

Когда нам приходится иметь дело с простыми наборами данных и/или с данными, которые
относительно постоянны во времени, подход REST по-прежнему остается наиболее эффективным и удобным.
идти. И наоборот, если нам нужно работать со сложными сценариями с быстро меняющимися данными, GraphQL может
решить некоторые болезненные недостатки REST и помочь нам создать более надежное, эффективное и удобное в обслуживании приложение.

# Package

```xml
<PackageReference Include="HotChocolate.AspNetCore" Version="13.9.9" />
<PackageReference Include="HotChocolate.AspNetCore.Authorization" Version="13.9.9" />
<PackageReference Include="HotChocolate.Data.EntityFramework" Version="13.9.9" />
```

# Program.cs

```Csharp

builder.Services.AddGraphQLServer()
    .AddAuthorization()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddFiltering()
    .AddSorting();

app.MapGraphQL("/api/graphql");

```



# Query

```Csharp
public class Query
{
    private readonly IMediator _mediator;

    public Query(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    
    /// <summary>
    /// Gets all Users.
    /// </summary>
    [Serial]
    [UsePaging]
    [UseFiltering]
    [UseSorting]
    public IQueryable<ApplicationUser> GetUsers([Service] ApplicationContext context)
        => context.Users;
    
    
    /// <summary>
    /// Gets all Users (with ApiResult and DTO support).
    /// </summary>
    [Serial]
    public async Task<ApiResult<Repository>> GetAllRepositories(
        
        [Service] ProjectStoreContext context,
        string? sortColumn = null,
        string? sortOrder = null,
        int pageIndex = 0,
        int pageSize = 10,
        string? filterColumn = null,
        string? filterQuery = null)
    {
        var response = await _mediator.Send(new RepositoryRequest(sortColumn,sortOrder,pageIndex,pageSize,filterColumn,filterQuery));

        return response.result;
    }
}
```

# Mutation

```Csharp
public class Mutation
{
    /// <summary>
    /// Add a new User
    /// </summary>
    ///
    [Serial]
    [Authorize]
    //[Authorize(Roles = new[] { "RegisteredUser" })]
    public async Task<ApplicationUser> AddUser(
        [Service] ApplicationContext context, UserDTO userDTO)
    {
        var user = new ApplicationUser()
        {  
            Password = userDTO.Password,
            Email = userDTO.Email
        };
        context.Users.Add(user);
        await context.SaveChangesAsync();
        return user;
    }

    /// <summary>
    /// Update an existing User
    /// </summary>
    [Serial]
    [Authorize(Roles = new[] { "RegisteredUser" })]
    public async Task<ApplicationUser> UpdateUser(
        [Service] ApplicationContext context, UserDTO userDTO)
    {
        var user = await context.Users
            .Where(c => c.Id == userDTO.Id)
            .FirstOrDefaultAsync();
        if (user == null)
            // todo: handle errors
            throw new NotSupportedException();
        user.Email = userDTO.Email;
        user.Password = userDTO.Password;
 
        context.Users.Update(user);
        await context.SaveChangesAsync();
        return user;
    }

    /// <summary>
    /// Delete a User
    ///
    /// /// </summary>
    [Serial]
    [Authorize(Roles = new[] { "Administrator" })]
    public async Task DeleteUser(
        [Service] ApplicationContext context, int id)
    {
        var User = await context.Users
            .Where(c => c.Id == id)
            .FirstOrDefaultAsync();
        if (User != null)
        {
            context.Users.Remove(User);
            await context.SaveChangesAsync();
        }
    }
}
```

# Запрос Muration

```graphql
mutation UpdateCity($user: UserDTOInput!) {
 addUser(userDTO: $user) {
 id
 email
 password
 }
}
```

