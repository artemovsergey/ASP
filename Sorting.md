# Сортировка

 ```Csharp

public enum SortState
{
    NameAsc,    // по имени по возрастанию
    NameDesc,   // по имени по убыванию
    AgeAsc, // по возрасту по возрастанию
    AgeDesc,    // по возрасту по убыванию
    CompanyAsc, // по компании по возрастанию
    CompanyDesc // по компании по убыванию
}
    
    
 public async Task<IActionResult> SortUsers(SortState sortOrder)
    {

        IQueryable<User>? users = db.Users.Include(x => x.Company);

        ViewData["NameSort"] = sortOrder == SortState.NameAsc ? SortState.NameDesc : SortState.NameAsc;
        ViewData["AgeSort"] = sortOrder == SortState.AgeAsc ? SortState.AgeDesc : SortState.AgeAsc;
        ViewData["CompSort"] = sortOrder == SortState.CompanyAsc ? SortState.CompanyDesc : SortState.CompanyAsc;


        users = sortOrder switch
        {
            SortState.NameDesc => users.OrderByDescending(s => s.Name),
            SortState.AgeAsc => users.OrderBy(s => s.Age),
            SortState.AgeDesc => users.OrderByDescending(s => s.Age),
            SortState.CompanyAsc => users.OrderBy(s => s.Company!.Name),
            SortState.CompanyDesc => users.OrderByDescending(s => s.Company!.Name),
            _ => users.OrderBy(s => s.Name),
        };



        return View(await users.AsNoTracking().ToListAsync());
    }
    
```

# Сортировка (view)

```html
    
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using Learning.Models
@model IEnumerable<User>

<h1>Список пользователей</h1>
<table>
    <tr>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewData["NameSort"]">
                Имя
            </a>
        </th>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewBag.AgeSort">
                Возраст
            </a>
        </th>
        <th>
            <a asp-action="SortUsers" asp-route-sortOrder="@ViewBag.CompSort">
                Компания
            </a>
        </th>
        @foreach (User u in Model)
        {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
        }
</table>
    
```

# helper for sorting
 
```Csharp

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.Routing;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Razor.TagHelpers;
using Learning.Models;

namespace Learning.TagHelpers
{
    public class SortHeaderTagHelper : TagHelper
    {
        public SortState Property { get; set; } // значение текущего свойства, для которого создается тег
        public SortState Current { get; set; }  // значение активного свойства, выбранного для сортировки
        public string? Action { get; set; }  // действие контроллера, на которое создается ссылка
        public bool Up { get; set; }    // сортировка по возрастанию или убыванию

        [ViewContext]
        [HtmlAttributeNotBound]
        public ViewContext ViewContext { get; set; } = null!;

        IUrlHelperFactory urlHelperFactory;
        public SortHeaderTagHelper(IUrlHelperFactory helperFactory)
        {
            urlHelperFactory = helperFactory;
        }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            IUrlHelper urlHelper = urlHelperFactory.GetUrlHelper(ViewContext);
            output.TagName = "a";
            string? url = urlHelper.Action(Action, new { sortOrder = Property });
            output.Attributes.SetAttribute("href", url);
            // если текущее свойство имеет значение CurrentSort
            if (Current == Property)
            {
                TagBuilder tag = new TagBuilder("i");
                tag.AddCssClass("glyphicon");

                if (Up == true)   // если сортировка по возрастанию
                    tag.AddCssClass("glyphicon-chevron-up");
                else   // если сортировка по убыванию
                    tag.AddCssClass("glyphicon-chevron-down");

                output.PreContent.AppendHtml(tag);
            }
        }
    }
}
    
```
