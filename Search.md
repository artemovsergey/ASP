# Поиск 
# Viewmodel
 
```Csharp
using Microsoft.AspNetCore.Mvc.Rendering;

namespace Learning.Models
{
    public class SearchUsersModel
    {
        public IEnumerable<User> Users { get; set; } = new List<User>();
        public string Name { get; set; }
    }
}
```
    
    
```Csharp
 
    public async Task<IActionResult> SearchUsers(string name)
    {
        IQueryable<User> users = db.Users.Include(p => p.Company);

        if (!string.IsNullOrEmpty(name))
        {
            users = users.Where(p => p.Name!.Contains(name));
        }


        SearchUsersModel searchModel = new SearchUsersModel
        {
            Users = users.ToList(),
            Name = name
        };


        return View(searchModel);
    }
    
```

# Поиск (view)
 
```html
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
    @using Learning.Models

    @model SearchUsersModel


<form method="get">
    <div>
        <label>Имя: </label>

        <input asp-for="Name" />

       

        <input type="submit" value="Поиск" />
    </div>
</form>



<h1>Список пользователей</h1>
<table>
    <tr>
        <th>
            
                Имя
         
        </th>
        <th>
            
                Возраст
        
        </th>
        <th>
            
                Компания
         
        </th>
        @foreach (User u in Model.Users)
        {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
        }
</table>    
```
