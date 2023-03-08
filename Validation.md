# Модель
# Валидация модели на стороне сервера

Важную роль в ASP.NET Core MVC играет валидация входных данных. Валидация позволяет проверить входные данные на наличие неправильных, корректных значений и должным образом обработать эти значения. Валидация модели в ASP.NET Core MVC базируется на общем механизме валидации, который имеется в .NET, однако ASP.NET Core MVC добавляет некоторую дополнительную инфраструктуру, которая облегчает процесс валидации.

Создать класс модели в папке Models

```csharp
using System.ComponentModel.DataAnnotations;

    public class User 
    {
        [Required(ErrorMessage = "Пожайлуста введите id пользователя")]
        public int Id { get; set; }
        
        [Required(ErrorMessage = "Пожайлуста введите имя пользователя")]
        public string Name { get; set; }

        [Required(ErrorMessage = "Пожайлуста введите возраст пользователя")]
        public int? Age { get; set; }
    }
```
# Проверка в контроллере

Валидация на стороне сервера, то есть в контроллере, осуществляется посредством помощью проверки свойства ModelState.IsValid. Объект ModelState сохраняет все значения, которые пользователь ввел для свойств модели, а также все ошибки, связанные с каждым свойством и с моделью в целом. Если в объекте ModelState имеются какие-нибудь ошибки, то свойство ModelState.IsValid возвратит false.


```Csharp

            if (ModelState.IsValid)
                return Content($"{person.Name} - {person.Age}");

            string errorMessages = "";
            // проходим по всем элементам в ModelState
            foreach (var item in ModelState)
            {
                // если для определенного элемента имеются ошибки
                if (item.Value.ValidationState == ModelValidationState.Invalid)
                {
                    errorMessages = $"{errorMessages}\nОшибки для свойства {item.Key}:\n";
                    // пробегаемся по всем ошибкам
                    foreach (var error in item.Value.Errors)
                    {
                        errorMessages = $"{errorMessages}{error.ErrorMessage}\n";
                    }
                }
            }
            return  View(person); // errorMessages;
            
```

# Добавление собственных валидаций 
При необходимости мы сами можем валидировать значения и добавлять в ModelState информацию об ошибках. Для этого применяется метод ModelState.AddModelError.

```Csharp

 if (person.Name == "admin" && person.Age == 25)
            {
                // добавляем ошибку уровня модели
                ModelState.AddModelError("", "Некорректные данные");
            }

            if (person.Age > 110 || person.Age < 0)
            {
                ModelState.AddModelError("Age", "Возраст должен находиться в диапазоне от 0 до 110");
            }
            if (person.Name?.Length < 3)
            {
                ModelState.AddModelError("Name", "Недопустимая длина строки. Имя должно иметь больше 2 символов");
            }

```

# Валидация на стороне клиента
Кроме валидации на стороне сервера с помощью свойства ModelState в ASP.NET Core MVC можно применять валидацию на стороне клиента, то есть на веб-странице. Валидация на стороне клиента позволяет уменьшить количество обращений к серверу и произвести все действия по проверке значений непосредственно при вводе данных.

# Подключение скирптов
```html
<script src="https://ajax.aspnetcdn.com/ajax/jQuery/jquery-3.5.1.min.js"></script>
<script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.17.0/jquery.validate.min.js"></script>
<script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.10/jquery.validate.unobtrusive.min.js"></script>
```
Для применения скриптов в разметке должен быть tag-helper:
```Csharp
 <span asp-validation-for="Name" class="text-danger"></span>
```
Таким образом, валидация на стороне клиента позволит уменьшить нагрузку на сервер. Тем не менее стоит учитывать, что на стороне клиента по тем или иным причинам может быть отключен javascript, либо данные отправляются в среде, где эти скрипты не применяются. Соответственно валидация на стороне сервера по прежнему актуальна. Кроме того, на стороне сервера могут проводиться дополнительные проверки, которые не могут быть выполнены на стороне клиента.

# Аттрибуты валидации
**Замечание**: для свойств тех типов данных, которые не могут принимать значение null, нам в любом случае надо предоставить значение, как в случае со свойством Age. Поэтому для него использовать атрибут Required необязательно.

# Аттрибут проверки регулярного выражения
```Csharp
[RegularExpression(@"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}", ErrorMessage = "Некорректный адрес")]
public string? Email { get; set; }
```
```Csharp
[EmailAddress (ErrorMessage = "Некорректный адрес")]
public string? Email { get; set; }
```
# Атрибут StringLength
Чтобы пользователь не мог ввести очень длинный текст, применяется атрибут StringLength. Первым параметром в конструкторе атрибута идет максимальная допустимая длина строки. Именованные параметры, в частности MinimumLength и ErrorMessage, позволяют задать дополнительные опции отображения.

```Csharp
[StringLength(50, MinimumLength = 3, ErrorMessage = "Длина строки должна быть от 3 до 50 символов")]
    public string? Name { get; set; }
```
# Атрибут Range
Атрибут Range определяет минимальные и максимальные значения для числовых данных.
```Csharp
[Range(1, 110, ErrorMessage = "Недопустимый возраст")]
public int Age { get; set; }
```
# Атрибут Compare
Атрибут Compare гарантирует, что два свойства объекта модели имеют одно и то же значение. Если, например, надо, чтобы пользователь ввел пароль дважды:

```Csharp
[Required]
public string? Password { get; set; }
 
[Compare("Password", ErrorMessage = "Пароли не совпадают")]
public string? PasswordConfirm { get; set; }
```
# Атрибут Remote
Атрибут Remote из пространства имен Microsoft.AspNetCore.Mvc; для валидации свойства выполняет запрос на сервер к определенному методу контроллера. И если требуемый метод контроллера вернет значение false, то валидация не пройдена. Например:

```Csharp
[Remote(action: "CheckEmail", controller: "Home", ErrorMessage ="Email уже используется")]
public string Email { get; set; }
``

# ValidationSummaryTagHelper

ValidationSummaryTagHelper применяется для отображения сводки ошибок валидации. Он применяется к элементу <div> в виде атрибута asp-validation-summary:

```html
<div asp-validation-summary="ModelOnly"/>
```

В качестве значения атрибут asp-validation-summary принимает одно из значений перечисления ValidationSummary:

- None: ошибки валидации не отображаются

- ModelOnly: отображаются только ошибка валидации уровня модели, ошибки валидации для отдельных свойств не отображаются

- All: отображаются все ошибки валидации


# Стилизация валидаций
Когда происходит валидация, то при отображении ошибок соответствующим полям присваиваются определенные классы css:

- для блока ошибок, который генерируется хелпером ValidationSummaryTagHelper, при наличии ошибок устанавливается класс validation-summary-errors. Если ошибок нет, то данный блок не отображается

- для элемента <span>, который отображает ошибку для каждого отдельного поля и который генерируется хелпером ValidationTagHelper, при наличии ошибок устанавливается класс field-validation-error. Если ошибок нет, то данный элемент имеет класс field-validation-valid

- для поля ввода при наличии ошибок устанавливается класс field-validation-error. Если ошибок нет, то устанавливается класс valid

```html
<style>
.field-validation-error {
    color: #b94a48;
}
  
input.input-validation-error {
    border: 1px solid #b94a48;
}
input.valid {
    border: 1px solid #16a085;
}
 
.validation-summary-errors {
    color: #b94a48;
}
</style>
```

# Создание атрибута валидации    
Встроенные атрибуты валидации охватывают большую часть возможных ситуаций, тем не менее этих атрибутов может оказаться недостаточно, особенно в каких-то специфических сценариях. Однако .NET позволяет определять собственные атрибуты валидации со своей логикой.
    
Для создания атрибута валидации необходимо унаследовать класс атрибута от ValidationAttribute и переопределить его метод IsValid().    
    
```Csharp
public class PersonNameAttribute : ValidationAttribute
    {
        //массив для хранения допустимых имен
        string[] _names;
 
        public PersonNameAttribute(string[] names)
        {
            _names = names;
        }
        public override bool IsValid(object? value)
        {
            return value != null && _names.Contains(value);
        }
```
    
Чтобы применить логику валидации, надо переопределить метод IsValid, предоставленный базовым классом. Параметр этого метода как раз и представляет то значение, которое надо валидировать.    
    
```Csharp
[PersonName(new string[] {"Tom", "Sam", "Alice" }, ErrorMessage ="Недопустимое имя")]
```

# Самовалидация и IValidatableObject
    
Самовалидация представляет собой процесс, при котором модель запускает механизм валидации из себя самой. И сама инкапсулирует всю логику валидации.

Для этого класс модели должен реализовать интерфейс IValidatableObject:

    
```Csharp
   using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
 
public class Person : IValidatableObject
{
    public string? Name { get; set; }
 
    public string? Email { get; set; }
    public string? Password { get; set; }
    public int Age { get; set; }
 
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var errors = new List<ValidationResult>();
 
        if (string.IsNullOrWhiteSpace(Name))
        {
            errors.Add(new ValidationResult("Введите имя!", new List<string>() { "Name" }));
        }
        if (string.IsNullOrWhiteSpace(Email))
        {
            errors.Add(new ValidationResult("Введите электронный адрес!"));
        }
        if (Age < 0 || Age > 120)
        {
            errors.Add(new ValidationResult("Недопустимый возраст!"));
        }
 
        return errors;
    }
} 
```
    
В данном случае нам надо реализовать метод Validate() и возвратить коллекцию объектов ValidationResult, которые и будут содержать все ошибки валидации. Если в конструктор ValidationResult передается только сообщение об ошибке, тогда данная ошибка будет относиться ко всей модели в целом. Однако с помощью второго параметра можно указать конкретный список свойств модели, к которым относится ошибка. То есть в данном случае ошибки для свойств Email и Age будут считаться ошибками для всей модели, а ошибка для Name будет ошибкой только этого конкретного свойства:    
    

# Аннотации данных    
Кроме атрибутов валидации модели также могут иметь дополнительные атрибуты, которые называются аннотации данных и которые располагаются в пространстве имен System.ComponentModel.DataAnnotations. Эти атрибуты определяют различные правила для отображения свойств модели.
 
# Атрибут Display       
```Csharp
public class Person
    {
        [Display(Name="Имя и фамилия")]
        public string? Name { get; set; }
        [Display(Name = "Электронная почта")]
        public string? Email { get; set; }
        [Display(Name = "Домашняя страница")]
        public string? HomePage { get; set; }
        [Display(Name = "Пароль")]
        public string? Password { get; set; }
        [Display(Name = "Возраст")]
        public int Age { get; set; }
    }    
```

# DisplayFormat
Атрибут DisplayFormat позволяет задать формат отображения свойства. Например, пусть у нас будет в модели свойство по типу DateTime:
    
```Csharp
[Display(Name = "Дата рождения")]
[DisplayFormat(DataFormatString = "{0:dd.MM.yyyy}", ApplyFormatInEditMode = true)]
public DateTime DateOfBirth { get; set; } 
```

Атрибут DisplayFormat настраивает формат с помощью ряда свойств. Так, свойство DataFormatString указывает на сам формат. Например, в данном случае мы прописали в формате вывод дня, месяца и года даты рождения.

Свойство ApplyFormatInEditMode позволяет применять форматирование к свойству даже в режиме редактирования.   
