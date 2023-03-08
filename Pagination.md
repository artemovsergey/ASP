# Пагинация (view)
  
```html
   @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, Learning

@using Learning.Models

@model PageUsersModel

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" />


<style>
    .glyphicon {
        display: inline-block;
        padding: 0 5px;
    }

    .glyphicon-chevron-right:after {
        content: "\00BB";
    }

    .glyphicon-chevron-left:before {
        content: "\00AB";
    }
</style>



<h1>Список пользователей</h1>

<table class="table">
    <tr><th>Имя</th><th>Возраст</th><th>Компания</th></tr>
    @foreach (User u in Model.Users)
    {
        <tr><td>@u.Name</td><td>@u.Age</td><td>@u.Company?.Name</td></tr>
    }
</table>

<p>
    @if (Model.HasPreviousPage)
    {
        <a asp-action="PageUsers" asp-route-page="@(Model.PageNumber - 1)" class="glyphicon glyphicon-chevron-left">
            Назад
        </a>
    }
    @if (Model.HasNextPage)
    {
        <a asp-action="PageUsers" asp-route-page="@(Model.PageNumber + 1)" class="glyphicon glyphicon-chevron-right">
            Вперед
        </a>
    }
</p>

 
```
  
# helper for sorting, filter, search and pagination
  
```Csharp
   using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.Routing;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Razor.TagHelpers;
using MvcApp.Models;
 
namespace MvcApp.TagHelpers
{
    public class PageLinkTagHelper : TagHelper
    {
        private IUrlHelperFactory urlHelperFactory;
        public PageLinkTagHelper(IUrlHelperFactory helperFactory)
        {
            urlHelperFactory = helperFactory;
        }
        [ViewContext]
        [HtmlAttributeNotBound]
        public ViewContext ViewContext { get; set; } = null!;
        public PageViewModel? PageModel { get; set; }
        public string PageAction { get; set; } = "";
 
        [HtmlAttributeName(DictionaryAttributePrefix = "page-url-")]
        public Dictionary<string, object> PageUrlValues { get; set; } = new();
 
        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            if(PageModel == null) throw new Exception("PageModel is not set");
            IUrlHelper urlHelper = urlHelperFactory.GetUrlHelper(ViewContext);
            output.TagName = "div";
 
            // набор ссылок будет представлять список ul
            TagBuilder tag = new TagBuilder("ul");
            tag.AddCssClass("pagination");
 
            // формируем три ссылки - на текущую, предыдущую и следующую
            TagBuilder currentItem = CreateTag(PageModel.PageNumber, urlHelper);
 
            // создаем ссылку на предыдущую страницу, если она есть
            if (PageModel.HasPreviousPage)
            {
                TagBuilder prevItem = CreateTag(PageModel.PageNumber - 1, urlHelper);
                tag.InnerHtml.AppendHtml(prevItem);
            }
 
            tag.InnerHtml.AppendHtml(currentItem);
            // создаем ссылку на следующую страницу, если она есть
            if (PageModel.HasNextPage)
            {
                TagBuilder nextItem = CreateTag(PageModel.PageNumber + 1, urlHelper);
                tag.InnerHtml.AppendHtml(nextItem);
            }
            output.Content.AppendHtml(tag);
        }
 
        TagBuilder CreateTag(int pageNumber, IUrlHelper urlHelper)
        {
            TagBuilder item = new TagBuilder("li");
            TagBuilder link = new TagBuilder("a");
            if (pageNumber == PageModel?.PageNumber)
            {
                item.AddCssClass("active");
            }
            else
            {
                PageUrlValues["page"] = pageNumber;
                link.Attributes["href"] = urlHelper.Action(PageAction, PageUrlValues);
            }
            item.AddCssClass("page-item");
            link.AddCssClass("page-link");
            link.InnerHtml.Append(pageNumber.ToString());
            item.InnerHtml.AppendHtml(link);
            return item;
        }
    }
} 
```
 # Пагинация
# Viewmodel
    
```Csharp
  using Microsoft.AspNetCore.Mvc.Rendering;
using System.Collections.Generic;


namespace Learning.Models
{
    public class PageUsersModel
    {

        public IEnumerable<User> Users { get; set; }
        public int PageNumber { get; }
        public int TotalPages { get; }
        public bool HasPreviousPage => PageNumber > 1;
        public bool HasNextPage => PageNumber < TotalPages;

    

        public PageUsersModel(int count, int pageNumber, int pageSize)
        {
            PageNumber = pageNumber;
            TotalPages = (int)Math.Ceiling(count / (double)pageSize);
        }


        

    }
}

```
    
```Csharp
    public async Task<IActionResult> PageUsers(int page = 1)
    {

        int pageSize = 3;   // количество элементов на странице

        IQueryable<User> source = db.Users.Include(x => x.Company);

        var count = await source.CountAsync();
        var items = await source.Skip((page - 1) * pageSize).Take(pageSize).ToListAsync();




        PageUsersModel pageViewModel = new PageUsersModel(count, page, pageSize)
        {
            Users = items
        };



        return View(pageViewModel);
    }
```
       
