# Migration

```
Add-Migraion Init -Project "RusRoads.Application" -StartupProject "RusRoads.Application"
Update-database -Project "RusRoads.Application" -StartupProject "RusRoads.Application"
```

# Тестовые данные

```Csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    //seed categories
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 1, Name = "Belgium" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 2, Name = "Germany" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 3, Name = "Netherlands" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 4, Name = "USA" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 5, Name = "Japan" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 6, Name = "China" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 7, Name = "UK" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 8, Name = "France" });
    modelBuilder.Entity<Country>().HasData(new Country { CountryId = 9, Name = "Brazil" });

    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 1, JobCategoryName = "Pie research"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 2, JobCategoryName = "Sales"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 3, JobCategoryName = "Management"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 4, JobCategoryName = "Store staff"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 5, JobCategoryName = "Finance"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 6, JobCategoryName = "QA"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 7, JobCategoryName = "IT"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 8, JobCategoryName = "Cleaning"});
    modelBuilder.Entity<JobCategory>().HasData(new JobCategory(){JobCategoryId = 9, JobCategoryName = "Bakery"});

    modelBuilder.Entity<Employee>().HasData(new Employee
    {
        EmployeeId = 1,
        CountryId = 1,
        MaritalStatus = MaritalStatus.Single,
        BirthDate = new DateTime(1979, 1, 16),
        City = "Brussels",
        Email = "bethany@bethanyspieshop.com",
        FirstName = "Bethany",
        LastName = "Smith",
        Gender = Gender.Female,
        PhoneNumber = "324777888773",
        Smoker = false,
        Street = "Grote Markt 1",
        Zip = "1000",
        JobCategoryId = 1,
        Comment = "Lorem Ipsum",
        ExitDate = null,
        JoinedDate = new DateTime(2015, 3, 1),
        Latitude = 50.8503, 
        Longitude = 4.3517
    });
}
```

# Генерация контроллеров и конечных точек
Если класс контекста лежит в другом проекте, то рядом с классом контекста создаем класс

```Csharp
public class ExampleContextFactory : IDesignTimeDbContextFactory<ExampleContext>
{
    public ExampleContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<ExampleContext>();
        optionsBuilder.UseNpgsql("Host=localhost;Port=5432;Database=Example;Username=postgres;Password=root");

        return new ExampleContext(optionsBuilder.Options);
    }
}
```

# AsNoTracking
API GetBooks возвращает коллекцию объектов Book. Ни одна из этих сущностей не изменяется. EF Core по умолчанию записывает все, что происходит с сущностями, которые он возвращает из запросов или добавляет в контекст. Эта функция, называемая отслеживанием изменений, является основной частью того, что делает EF Core таким мощным. Например, если код изменяет свойство Title объекта Book, EF Core записывает это изменение, а при вызове SaveChanges преобразует его в SQL-оператор UPDATE. Когда вы запрашиваете данные и не хотите вносить изменения, отключите отслеживание изменений, чтобы предотвратить случайные изменения, а также сэкономить память и процессор. На уровне запроса можно отключить отслеживание изменений с помощью метода расширения AsNoTracking.


# IAsyncEnumerable.
В отличие от IEnumerable, который возвращает синхронный перечислитель. Синхронные перечислители позволяют избежать блокировки потоков, сначала получая все записи, а затем перечисляя их. Если вы хотите отправлять записи обратно в ответ по мере их получения из базы данных, не блокируя поток, IEnumerable не подойдет. IAsyncEnumerable позволяет действию начать отправку данных ответа, как только запрос базы данных начнет получать записи из базы данных. Без IAsyncEnumerable действие должно ждать отправки ответа, пока не будут получены все записи из базы данных и не будет готово полное содержимое ответа. Хотя это действие не имеет особых преимуществ для нашего примера, оно полезно для сервисов, которые работают с большим количеством данных (или медленными запросами).

