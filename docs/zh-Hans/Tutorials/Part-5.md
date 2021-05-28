# Web 应用程序开发教程 - 第 5 部分：授权

## 关于此教程

在此教程系列中，您将构建一个名为 ABP 的 Web 应用程序。此应用程序用于管理书籍及其作者的列表。它使用以下技术开发：`Acme.BookStore`

- **实体框架核心**作为ORM提供商。
- **MVC/剃刀页**作为UI框架。

本教程分为以下部分：

- [第 1 部分：创建服务器端](https://docs.abp.io/en/abp/latest/Tutorials/Part-1)
- [第 2 部分：图书列表页](https://docs.abp.io/en/abp/latest/Tutorials/Part-2)
- [第 3 部分：创建、更新和删除书籍](https://docs.abp.io/en/abp/latest/Tutorials/Part-3)
- [第 4 部分：集成测试](https://docs.abp.io/en/abp/latest/Tutorials/Part-4)
- **第 5 部分：授权（本部分）**
- [第 6 部分：作者：域层](https://docs.abp.io/en/abp/latest/Tutorials/Part-6)
- [第 7 部分：作者：数据库集成](https://docs.abp.io/en/abp/latest/Tutorials/Part-7)
- [第 8 部分：作者：应用层](https://docs.abp.io/en/abp/latest/Tutorials/Part-8)
- [第 9 部分：作者：用户界面](https://docs.abp.io/en/abp/latest/Tutorials/Part-9)
- [第10部分：书与作者关系](https://docs.abp.io/en/abp/latest/Tutorials/Part-10)

### 下载源代码

此教程基于您的**用户界面**和**数据库**首选项具有多个版本。我们准备了要下载的源代码的几个组合：

- [MVC（剃刀页）UI与EF核心](https://github.com/abpframework/abp-samples/tree/master/BookStore-Mvc-EfCore)
- [带EF核心的布拉佐尔用户界面](https://github.com/abpframework/abp-samples/tree/master/BookStore-Blazor-EfCore)
- [角UI与蒙哥德布](https://github.com/abpframework/abp-samples/tree/master/BookStore-Angular-MongoDb)

### 视频教程

这部分也被记录为视频教程，并在**[YouTube上发布](https://www.youtube.com/watch?v=1WsfMITN_Jk&list=PLsNclT2aHJcPNaCf7Io3DbMN6yAk_DgWJ&index=5)**。

## 权限

ABP 框架提供基于 ASP.NET 核心[的授权基础架构的授权](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/introduction)[系统](https://docs.abp.io/en/abp/latest/Authorization)。在标准授权基础结构之上添加的一个主要功能是允许定义权限并启用/禁用每个角色、用户或客户端的**权限系统**。

### 权限名称

权限必须具有唯一的名称 （a ）。最好的方法是将其定义为 a，以便我们可以重复使用权限名称。`string``const`

打开项目内的类（在文件夹中），并更改下文所示的内容：`BookStorePermissions``Acme.BookStore.Application.Contracts``Permissions`

```csharp
namespace Acme.BookStore.Permissions
{
    public static class BookStorePermissions
    {
        public const string GroupName = "BookStore";

        public static class Books
        {
            public const string Default = GroupName + ".Books";
            public const string Create = Default + ".Create";
            public const string Edit = Default + ".Edit";
            public const string Delete = Default + ".Delete";
        }
    }
}
```

C#

复制

这是定义权限名称的分层方式。例如，"创建图书"权限名称定义为。ABP 不会强迫您构建一个结构，但我们发现这种方式很有用。`BookStore.Books.Create`

### 权限定义

在使用权限之前，您应该定义权限。

打开项目内的类（在文件夹中），并更改下文所示的内容：`BookStorePermissionDefinitionProvider``Acme.BookStore.Application.Contracts``Permissions`

```csharp
using Acme.BookStore.Localization;
using Volo.Abp.Authorization.Permissions;
using Volo.Abp.Localization;

namespace Acme.BookStore.Permissions
{
    public class BookStorePermissionDefinitionProvider : PermissionDefinitionProvider
    {
        public override void Define(IPermissionDefinitionContext context)
        {
            var bookStoreGroup = context.AddGroup(BookStorePermissions.GroupName, L("Permission:BookStore"));

            var booksPermission = bookStoreGroup.AddPermission(BookStorePermissions.Books.Default, L("Permission:Books"));
            booksPermission.AddChild(BookStorePermissions.Books.Create, L("Permission:Books.Create"));
            booksPermission.AddChild(BookStorePermissions.Books.Edit, L("Permission:Books.Edit"));
            booksPermission.AddChild(BookStorePermissions.Books.Delete, L("Permission:Books.Delete"));
        }

        private static LocalizableString L(string name)
        {
            return LocalizableString.Create<BookStoreResource>(name);
        }
    }
}
```

C#

复制

本类定义**了权限组**（在UI上对组权限，如下所示）和本组中的**4 个权限**。此外，**创建**、**编辑**和**删除**是权限的子项。**只有在选择家长时，**才能选择儿童权限。`BookStorePermissions.Books.Default`

最后，编辑本地化文件（在项目文件夹下）以定义上述使用的本地化密钥：`en.json``Localization/BookStore``Acme.BookStore.Domain.Shared`

```json
"Permission:BookStore": "Book Store",
"Permission:Books": "Book Management",
"Permission:Books.Create": "Creating new books",
"Permission:Books.Edit": "Editing the books",
"Permission:Books.Delete": "Deleting the books"
```

杰森

复制

> Localization key names are arbitrary and there is no forcing rule. But we prefer the convention used above.

### Permission Management UI

Once you define the permissions, you can see them on the **permission management modal**.

Go to the *Administration -> Identity -> Roles* page, select *Permissions* action for the admin role to open the permission management modal:

[![bookstore-permissions-ui](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-permissions-ui.png)](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-permissions-ui.png)

Grant the permissions you want and save the modal.

> **Tip**: New permissions are automatically granted to the admin role if you run the application.`Acme.BookStore.DbMigrator`

## Authorization

Now, you can use the permissions to authorize the book management.

### Application Layer & HTTP API

Open the class and add set the policy names as the permission names defined above:`BookAppService`

```csharp
using System;
using Acme.BookStore.Permissions;
using Volo.Abp.Application.Dtos;
using Volo.Abp.Application.Services;
using Volo.Abp.Domain.Repositories;

namespace Acme.BookStore.Books
{
    public class BookAppService :
        CrudAppService<
            Book, //The Book entity
            BookDto, //Used to show books
            Guid, //Primary key of the book entity
            PagedAndSortedResultRequestDto, //Used for paging/sorting
            CreateUpdateBookDto>, //Used to create/update a book
        IBookAppService //implement the IBookAppService
    {
        public BookAppService(IRepository<Book, Guid> repository)
            : base(repository)
        {
            GetPolicyName = BookStorePermissions.Books.Default;
            GetListPolicyName = BookStorePermissions.Books.Default;
            CreatePolicyName = BookStorePermissions.Books.Create;
            UpdatePolicyName = BookStorePermissions.Books.Edit;
            DeletePolicyName = BookStorePermissions.Books.Delete;
        }
    }
}
```

C#

Copy

向构造器添加代码。基地自动在CRUD操作中使用这些权限。这使得**应用程序服务**安全，但也使**HTTP API**安全，因为此服务会自动用作 HTTP API，如前所述（参见[自动 API 控制器](https://docs.abp.io/en/abp/latest/API/Auto-API-Controllers)）。`CrudAppService`

> 稍后，在开发作者管理功能时，您将看到声明授权（使用属性）。`[Authorize(...)]`

### 剃刀页

虽然保护 HTTP API – 应用程序服务阻止未经授权的用户使用服务，但他们仍然可以导航到图书管理页面。虽然当页面向服务器发出第一个 AJAX 呼叫时，它们将获得授权例外，但我们也应该授权该页面以获得更好的用户体验和安全性。

打开并添加方法内的以下代码块：`BookStoreWebModule``ConfigureServices`

```csharp
Configure<RazorPagesOptions>(options =>
{
    options.Conventions.AuthorizePage("/Books/Index", BookStorePermissions.Books.Default);
    options.Conventions.AuthorizePage("/Books/CreateModal", BookStorePermissions.Books.Create);
    options.Conventions.AuthorizePage("/Books/EditModal", BookStorePermissions.Books.Edit);
});
```

C#

复制

现在，未经授权的用户被重定向到**登录页面**。

#### 隐藏新书按钮

图书管理页面有一*个新书*按钮，如果当前用户没有*图书创建*权限，该按钮应不可见。

[![bookstore-new-book-button-small](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-new-book-button-small.png)](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-new-book-button-small.png)

打开文件并更改下文所示的内容：`Pages/Books/Index.cshtml`

```html
@page
@using Acme.BookStore.Localization
@using Acme.BookStore.Permissions
@using Acme.BookStore.Web.Pages.Books
@using Microsoft.AspNetCore.Authorization
@using Microsoft.Extensions.Localization
@model IndexModel
@inject IStringLocalizer<BookStoreResource> L
@inject IAuthorizationService AuthorizationService
@section scripts
{
    <abp-script src="/Pages/Books/Index.js"/>
}

<abp-card>
    <abp-card-header>
        <abp-row>
            <abp-column size-md="_6">
                <abp-card-title>@L["Books"]</abp-card-title>
            </abp-column>
            <abp-column size-md="_6" class="text-right">
                @if (await AuthorizationService.IsGrantedAsync(BookStorePermissions.Books.Create))
                {
                    <abp-button id="NewBookButton"
                                text="@L["NewBook"].Value"
                                icon="plus"
                                button-type="Primary"/>
                }
            </abp-column>
        </abp-row>
    </abp-card-header>
    <abp-card-body>
        <abp-table striped-rows="true" id="BooksTable"></abp-table>
    </abp-card-body>
</abp-card>
```

赫特姆

复制

- 添加到授权服务的访问。`@inject IAuthorizationService AuthorizationService`
- 用于检查图书创建权限，以便有条件地呈现*新书*按钮。`@if (await AuthorizationService.IsGrantedAsync(BookStorePermissions.Books.Create))`

### 爪哇脚本侧

图书管理页面中的图书表每行都有一个操作按钮。操作按钮包括*编辑*和*删除*操作：

[![bookstore-edit-delete-actions](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-edit-delete-actions.png)](https://raw.githubusercontent.com/abpframework/abp/rel-4.3/docs/en/Tutorials/images/bookstore-edit-delete-actions.png)

如果当前用户未授予相关权限，则应隐藏操作。数据表行操作有一个选项，可以设置为隐藏操作项。`visible``false`

打开项目内部，并在操作中添加以下选项：`Pages/Books/Index.js``Acme.BookStore.Web``visible``Edit`

```js
{
    text: l('Edit'),
    visible: abp.auth.isGranted('BookStore.Books.Edit'), //CHECK for the PERMISSION
    action: function (data) {
        editModal.open({ id: data.record.id });
    }
}
```

JavaScript

Copy

Do same for the action:`Delete`

```js
visible: abp.auth.isGranted('BookStore.Books.Delete')
```

JavaScript

Copy

- `abp.auth.isGranted(...)` is used to check a permission that is defined before.
- `visible` could also be get a function that returns a if the value will be calculated later, based on some conditions.`bool`

### Menu Item

Even we have secured all the layers of the book management page, it is still visible on the main menu of the application. We should hide the menu item if the current user has no permission.

Open the class, find the code block below:`BookStoreMenuContributor`

```csharp
context.Menu.AddItem(
    new ApplicationMenuItem(
        "BooksStore",
        l["Menu:BookStore"],
        icon: "fa fa-book"
    ).AddItem(
        new ApplicationMenuItem(
            "BooksStore.Books",
            l["Menu:Books"],
            url: "/Books"
        )
    )
);
```

C#

Copy

并将此代码块替换为以下代码块：

```csharp
var bookStoreMenu = new ApplicationMenuItem(
    "BooksStore",
    l["Menu:BookStore"],
    icon: "fa fa-book"
);

context.Menu.AddItem(bookStoreMenu);

//CHECK the PERMISSION
if (await context.IsGrantedAsync(BookStorePermissions.Books.Default))
{
    bookStoreMenu.AddItem(new ApplicationMenuItem(
        "BooksStore.Books",
        l["Menu:Books"],
        url: "/Books"
    ));
}
```

C#

复制

您还需要将关键字添加到该方法并重新排列返回值。最后一堂课应如下：`async``ConfigureMenuAsync``BookStoreMenuContributor`

```csharp
using System.Threading.Tasks;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Localization;
using Acme.BookStore.Localization;
using Acme.BookStore.MultiTenancy;
using Acme.BookStore.Permissions;
using Volo.Abp.TenantManagement.Web.Navigation;
using Volo.Abp.UI.Navigation;

namespace Acme.BookStore.Web.Menus
{
    public class BookStoreMenuContributor : IMenuContributor
    {
        public async Task ConfigureMenuAsync(MenuConfigurationContext context)
        {
            if (context.Menu.Name == StandardMenus.Main)
            {
                await ConfigureMainMenuAsync(context);
            }
        }

        private async Task ConfigureMainMenuAsync(MenuConfigurationContext context)
        {
            if (!MultiTenancyConsts.IsEnabled)
            {
                var administration = context.Menu.GetAdministration();
                administration.TryRemoveMenuItem(TenantManagementMenuNames.GroupName);
            }

            var l = context.GetLocalizer<BookStoreResource>();

            context.Menu.Items.Insert(0, new ApplicationMenuItem("BookStore.Home", l["Menu:Home"], "~/"));

            var bookStoreMenu = new ApplicationMenuItem(
                "BooksStore",
                l["Menu:BookStore"],
                icon: "fa fa-book"
            );

            context.Menu.AddItem(bookStoreMenu);

            //CHECK the PERMISSION
            if (await context.IsGrantedAsync(BookStorePermissions.Books.Default))
            {
                bookStoreMenu.AddItem(new ApplicationMenuItem(
                    "BooksStore.Books",
                    l["Menu:Books"],
                    url: "/Books"
                ));
            }
        }
    }
}
```

C#

复制

## 下一部分

请参阅本教程[的下一部分](https://docs.abp.io/en/abp/latest/Tutorials/Part-6)。
