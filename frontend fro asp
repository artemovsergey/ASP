# Настройка webpack

Затем создайте новый каталог ```ClientApp``` в корне вашего проекта MVC. Внутри этого нового каталога создайте файл с именем ```package.json```.

    
## package.json
    
```json
    
{
  "name": "myapp-client-bundle",
  "version": "1.0.0",
  "description": "This is client-side scripts bundle for MyApp",
  "private": true,

  "scripts": {
    "build": "webpack --mode=development",
    "build:prod": "webpack --mode=production",
    "watch": "webpack --watch"
  },

  "devDependencies": {

    "ts-loader": "^9.2.5",
    "typescript": "^4.4.3",
    "webpack": "^5.52.1",
    "webpack-cli": "^4.8.0",
    "css-loader": "^6.7.1",

    "style-loader": "^3.3.1",
    "mini-css-extract-plugin": "^2.6.0"

  },
  "dependencies": {

    "@popperjs/core": "^2.11.2",
    "jquery": "^3.6.0",
    "jquery-validation": "^1.19.3",
    "jquery-validation-unobtrusive": "^3.2.12",
    "bootstrap": "5.2.0"
  }
}
    
```

Выполнить установку пакетов npm
```npm install```
    
 Появится новый ```node_modules``` каталог, содержащий огромное количество файлов JS и CSS. Считайте, что этот каталог похож на каталоги binи obj (или, что еще лучше, на кеш пакетов NuGet). Хотя он не содержит двоичных файлов, его содержимое по-прежнему представляет собой набор зависимостей, на которые мы ссылаемся из нашего собственного кода.
    
Создаем каталог ```ClinetApp``` в нем создаем папку ```src```, в ней создаем файлы, например ```index.js```.
    
```js
    
// JS Dependencies: Popper, Bootstrap & JQuery
import '@popperjs/core';
import 'bootstrap';
import 'jquery';
// Using the next two lines is like including partial view _ValidationScriptsPartial.cshtml
import 'jquery-validation';
import 'jquery-validation-unobtrusive';

// CSS Dependencies: Bootstrap
import 'bootstrap/dist/css/bootstrap.css';

// Custom JS imports
// ... none at the moment

// Custom CSS imports
import '../css/site.css';

console.log('The \'site\' bundle has been loaded!');
    
```

Строки со 2 по 9 будут импортировать код изнутри node_modules. Мы импортируем Javascript, Popper и CSS Bootstrap, а также несколько библиотек JQuery, которые используются сценариями проверки ASP.NET Core.
Все это делается с помощью модулей ECMAScript 6 — современного подхода к импорту JS-файлов из других JS-файлов или даже CSS-файлов из JS-файлов.

 
## webpack.config.js

 Создайте новый файл с именем ```webpack.config.js``` и поместите его в ClientAppкаталог.
    
```js

const path = require('path');
MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
    entry: {
        site: './src/js/site.js',
        bootstrap_js: './src/js/bootstrap_js.js',
        validation: './src/js/validation.js',
        index: './src/js/index.js',
        indexTs: './src/ts/index.ts'
    },

    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
    },

    output: {

        library: {
            name: 'MYAPP',
            type: 'var'
        },

        filename: '[name].entry.js',
        path: path.resolve(__dirname, '..', 'wwwroot', 'dist')
    },
    devtool: 'source-map',
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.css$/,
                //use: ['style-loader', 'css-loader'],
                use: [{ loader: MiniCssExtractPlugin.loader }, 'css-loader']
            },
            {
                test: /\.(eot|woff(2)?|ttf|otf|svg)$/i,
                type: 'asset'
            },

            {
                test: /\.tsx?$/,
                use: 'ts-loader',
                exclude: /node_modules/,
            }

        ]
    },

     plugins: [
     new MiniCssExtractPlugin({ filename: "[name].css"})
     ]

};
    
```

## tsconfig.json
 
```json
 
{
  "compilerOptions": {
    "outDir": "./dist/",
    "noImplicitAny": true,
    "module": "es6",
    "target": "es5",
    "allowJs": true,
    "moduleResolution": "node"
  }
}
    
```
    
## index.ts
  
```ts
export * from './hello';  
```
    
## hello.ts
  
```ts
export namespace funcs {
    export function hello(): void {
        const message = 'Hello world!';
        console.log(message);
    }
}
```    
 
Можно подключать
    
```js
<script src="/dist/indexTs.entry.js"></script>


<script>
MYAPP.funcs.hello();
</script>
```
    
    
Далее ```npm run build```
   

Подключить файл в макете:

```html
<script src="~/dist/site.entry.js" defer></script>
```

Скрипт для валидации asp форм:
 
```html
<script src="~/dist/validation.entry.js" defer></script>   
```
    
## Скрипт выпадающий список
  
```html
<div class="dropdown">
    <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
        Dropdown button
    </button>
    <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
        <li><a class="dropdown-item" href="#">Action</a></li>
        <li><a class="dropdown-item" href="#">Another action</a></li>
        <li><a class="dropdown-item" href="#">Something else here</a></li>
    </ul>
</div>
```
 
```
 <script src="~/dist/bootstrap_js.entry.js" defer></script>
```
 
Замечание: можно применить секции
  
```html
 @await RenderSectionAsync("Scripts", required: false)
```
 
```Csharp
    
    @section Scripts
    {
     <script src="~/dist/bootstrap_js.entry.js" defer></script>
    }
    
```

Для наблюдения за изменениями:
    
```npm run watch```
    
Для сборки webpack при сборке dotnet можно настроить csproject
    

```Csharp
<Project Sdk="Microsoft.NET.Sdk.Web">

     <PropertyGroup>
         <TargetFramework>net6.0</TargetFramework>
         <Nullable>disable</Nullable>
         <ImplicitUsings>enable</ImplicitUsings>
        <IsPackable>false</IsPackable>
        <MpaRoot>ClientApp\</MpaRoot>
       <WWWRoot>wwwroot\</WWWRoot>
        <DefaultItemExcludes>$(DefaultItemExcludes);$(MpaRoot)node_modules\**</DefaultItemExcludes>
     </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.SpaServices.Extensions" Version="6.0.3"/>
    </ItemGroup>

    <ItemGroup>
        <!-- Don't publish the MPA source files, but do show them in the project files list -->
        <Content Remove="$(MpaRoot)**"/>
        <None Remove="$(MpaRoot)**"/>
        <None Include="$(MpaRoot)**" Exclude="$(MpaRoot)node_modules\**"/>
    </ItemGroup>

    <Target Name="NpmInstall" BeforeTargets="Build" Condition=" '$(Configuration)' == 'Debug' And !Exists('$(MpaRoot)node_modules') ">
        <!-- Ensure Node.js is installed -->
        <Exec Command="node --version" ContinueOnError="true">
            <Output TaskParameter="ExitCode" PropertyName="ErrorCode"/>
        </Exec>
        <Error Condition="'$(ErrorCode)' != '0'" Text="Node.js is required to build and run this project. To continue, please install Node.js from https://nodejs.org/, and then restart your command prompt or IDE."/>
        <Message Importance="high" Text="Restoring dependencies using 'npm'. This may take several minutes..."/>
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm install"/>
    </Target>

    <Target Name="NpmRunBuild" BeforeTargets="Build" DependsOnTargets="NpmInstall">
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm run build"/>
    </Target>

    <Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish">
        <!-- As part of publishing, ensure the JS resources are freshly built in production mode -->
        <Exec WorkingDirectory="$(MpaRoot)" Command="npm install"/>
       <Exec WorkingDirectory="$(MpaRoot)" Command="npm run build"/>

        <!-- Include the newly-built files in the publish output -->
       <ItemGroup>
            <DistFiles Include="$(WWWRoot)dist\**"/>
            <ResolvedFileToPublish Include="@(DistFiles->'%(FullPath)')" Exclude="@(ResolvedFileToPublish)">
                <RelativePath>%(DistFiles.Identity)</RelativePath>
                <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
                <ExcludeFromSingleFile>true</ExcludeFromSingleFile>
           </ResolvedFileToPublish>
        </ItemGroup>
    </Target>

    <Target Name="NpmClean" BeforeTargets="Clean">
        <RemoveDir Directories="$(WWWRoot)dist"/>
        <RemoveDir Directories="$(MpaRoot)node_modules"/>
   </Target>

 </Project>
```
    
    
    
В результате:
- Мы ссылаемся на наши библиотеки с определенным номером версии
- Зависимости не размещаются внутри дерева проекта
- В итоге мы получим прирост производительности (в стандартном шаблоне MVC на все Bootstrap и JQuery ссылаются со всех страниц).
Наша система сборки расширяема: Хотите Sass? Без проблем! Хотите использовать самые последние функции ECMAScript? Ты понял! Нужна минификация или обфускация? Без проблем!
