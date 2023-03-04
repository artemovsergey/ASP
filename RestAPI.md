## Program.cs для API

```Csharp
using Microsoft.EntityFrameworkCore;
using WebAPILearning.Data;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

/*
 * 
В ASP.NET Core службы (такие как контекст базы данных) должны быть зарегистрированы с помощью 
контейнера внедрения зависимостей. Контейнер предоставляет службу контроллерам.
 
 */

string connection = builder.Configuration.GetConnectionString("PostgreSQL");
builder.Services.AddDbContext<UserContext>(options => options.UseNpgsql(connection));

//builder.Services.AddDbContext<UserContext>(opt =>opt.UseInMemoryDatabase("UsersDatabase"));
// Добавляет контекст базы данных в контейнер внедрения зависимостей.
// Указывает, что контекст базы данных будет использовать базу данных в памяти.

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();


```

# ReestAPI контроллер для модели User

```Csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using WebAPI.Models;

namespace WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UsersController : ControllerBase
    {
        private readonly UserContext _context;

        public UsersController(UserContext context)
        {
            _context = context;
        }



        // GET: api/Users
        [HttpGet]
        public async Task<ActionResult<IEnumerable<User>>> GetUsers()
        {
            return await _context.Users.ToListAsync();
        }



        // GET: api/Users/5
        [HttpGet("{id}")]
        public async Task<ActionResult<User>> GetUser(int id)
        {
            var user = await _context.Users.FindAsync(id);

            if (user == null)
            {
                return NotFound();
            }

            return user;
        }


        // PUT: api/TodoItems/5
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateUsers(int id, User user)
        {
            if (id != user.Id)
            {
                return BadRequest();
            }

            var current_user = await _context.Users.FindAsync(id);
            if (current_user == null)
            {
                return NotFound();
            }

            current_user.Name = user.Name;
            current_user.Age = user.Age;

            try
            {
                await _context.SaveChangesAsync();
            }
            catch
            {
                return NotFound();
            }

            return NoContent();
        }


        // DELETE: api/TodoItems/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteTodoItem(int id)
        {
            var todoItem = await _context.Users.FindAsync(id);
            if (todoItem == null)
            {
                return NotFound();
            }

            _context.Users.Remove(todoItem);
            await _context.SaveChangesAsync();

            return NoContent();
        }


        // POST: api/TodoItems
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPost]
        public async Task<ActionResult<User>> CreateTodoItem(User user)
        {
            var user1 = new User
            {
                Age = user.Age,
                Name = user.Name
            };

            _context.Users.Add(user1);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user1);



        }
    }
}
```

# UserContext

```Csharp
using Microsoft.EntityFrameworkCore;
using WebAPILearning.Models;

namespace WebAPILearning.Data
{
    public class UserContext : DbContext
    {
        public UserContext()
        {
            Database.EnsureCreated();    
        }

        public UserContext(DbContextOptions<UserContext> options) : base(options)
        {
            Database.EnsureCreated();
        }

        public DbSet<User> Users { get; set; } 
    }
}

```

# appsettings.json

```json
{
  "ConnectionStrings": {

    "MSSQL": "Server=WIN-PO9SVP3KRMT\\MSSQLSERVER01;Database=SportStore;Trusted_Connection=True;MultipleActiveResultSets=true",
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SportStore;Trusted_Connection=True;MultipleActiveResultSets=true",
    "PostgreSQL": "Host=localhost;Port=5432;Database=SportStore1;Username=postgres;Password=root",
    "MySQL": "server=localhost;user=root;password=root;database=SportStore;",
    "SQLite": "Data Source=SportStore.db"

  },

  "Logging": {
    "LogLevel": {
      "Default": "None",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "AllowedHosts": "*"
}

```



