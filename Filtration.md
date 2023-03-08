# Фильтрация 
# Viewmodel
 
```Csharp
    
using Microsoft.AspNetCore.Mvc.Rendering;
using System.Collections.Generic;


namespace Learning.Models
{
    public class FilterViewModel
    {
        public FilterViewModel(List<Company> companies, int company, string name)
        {
            // устанавливаем начальный элемент, который позволит выбрать всех
            companies.Insert(0, new Company { Name = "Все", Id = 0 });
            Companies = new SelectList(companies, "Id", "Name", company);
            SelectedCompany = company;
            SelectedName = name;
        }
        public SelectList Companies { get; } // список компаний
        public int SelectedCompany { get; } // выбранная компания
        public string SelectedName { get; } // введенное имя
    }
}

    
```
       
```Csharp
    
    public async Task<IActionResult> FilterUsers(int company)
    {

        IQueryable<User> users = db.Users.Include(p => p.Company);

        if (company != null && company != 0)
        {
            users = users.Where(u => u.CompanyId == company);
        }



        List<Company> companies = db.Companies.ToList();
        // устанавливаем начальный элемент, который позволит выбрать всех
        companies.Insert(0, new Company { Name = "Все", Id = 0 });



        FilterUsersModel filterModel = new FilterUsersModel
        {
            Users = users.ToList(),
            Companies = new SelectList(companies, "Id", "Name", company),
            
        };


        return View(filterModel);
    }
```

# Фильтрация (view)
 
```htnl
   
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using Learning.Models

@model FilterUsersModel


<form method="get">
    <div>
        <label>Компания: </label>
        <select name="company" asp-items="Model.Companies"></select>

        <input type="submit" value="Фильтр" />
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
