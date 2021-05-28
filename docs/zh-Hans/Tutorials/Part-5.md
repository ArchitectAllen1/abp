用户界面

MVC / 剃刀页
数据库

实体框架核心
Web 应用程序开发教程 - 第 5 部分：授权
关于此教程
在此教程系列中，您将构建一个名为 ABP 的 Web 应用程序。此应用程序用于管理书籍及其作者的列表。它使用以下技术开发：Acme.BookStore

实体框架核心作为ORM提供商。
MVC/剃刀页作为UI框架。
本教程分为以下部分：

第 1 部分：创建服务器端
第 2 部分：图书列表页
第 3 部分：创建、更新和删除书籍
第 4 部分：集成测试
第 5 部分：授权（本部分）
第 6 部分：作者：域层
第 7 部分：作者：数据库集成
第 8 部分：作者：应用层
第 9 部分：作者：用户界面
第10部分：书与作者关系
下载源代码
此教程基于您的用户界面和数据库首选项具有多个版本。我们准备了要下载的源代码的几个组合：

MVC（剃刀页）UI与EF核心
带EF核心的布拉佐尔用户界面
角UI与蒙哥德布
视频教程
这部分也被记录为视频教程，并在YouTube上发布。

权限
ABP 框架提供基于 ASP.NET 核心的授权基础架构的授权系统。在标准授权基础结构之上添加的一个主要功能是允许定义权限并启用/禁用每个角色、用户或客户端的权限系统。

权限名称
权限必须具有唯一的名称 （a ）。最好的方法是将其定义为 a，以便我们可以重复使用权限名称。stringconst

打开项目内的类（在文件夹中），并更改下文所示的内容：BookStorePermissionsAcme.BookStore.Application.ContractsPermissions

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
C#
这是定义权限名称的分层方式。例如，"创建图书"权限名称定义为。ABP 不会强迫您构建一个结构，但我们发现这种方式很有用。BookStore.Books.Create

权限定义
在使用权限之前，您应该定义权限。

打开项目内的类（在文件夹中），并更改下文所示的内容：BookStorePermissionDefinitionProviderAcme.BookStore.Application.ContractsPermissions

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
C#
本类定义了权限组（在UI上对组权限，如下所示）和本组中的4 个权限。此外，创建、编辑和删除是权限的子项。只有在选择家长时，才能选择儿童权限。BookStorePermissions.Books.Default

最后，编辑本地化文件（在项目文件夹下）以定义上述使用的本地化密钥：en.jsonLocalization/BookStoreAcme.BookStore.Domain.Shared

"Permission:BookStore": "Book Store",
"Permission:Books": "Book Management",
"Permission:Books.Create": "Creating new books",
"Permission:Books.Edit": "Editing the books",
"Permission:Books.Delete": "Deleting the books"
杰森
本地化键名是任意的，没有强制规则。但我们更喜欢上面使用的约定。

权限管理用户界面
定义权限后，您可以在权限管理模式上查看它们。

转到"管理->身份->角色"页面，选择权限操作，以便管理员角色打开权限管理模式：

bookstore-permissions-ui

授予您想要的权限并保存模式。

提示：如果运行应用程序，将自动授予管理员角色新的权限。Acme.BookStore.DbMigrator

授权
现在，您可以使用权限授权图书管理。

应用层和高频API
打开类并添加将策略名称设置为上述定义的权限名称：BookAppService

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
C#
Added code to the constructor. Base automatically uses these permissions on the CRUD operations. This makes the application service secure, but also makes the HTTP API secure since this service is automatically used as an HTTP API as explained before (see auto API controllers).CrudAppService

You will see the declarative authorization, using the attribute, later while developing the author management functionality.[Authorize(...)]

Razor Page
While securing the HTTP API & the application service prevents unauthorized users to use the services, they can still navigate to the book management page. While they will get authorization exception when the page makes the first AJAX call to the server, we should also authorize the page for a better user experience and security.

Open the and add the following code block inside the method:BookStoreWebModuleConfigureServices

Configure<RazorPagesOptions>(options =>
{
    options.Conventions.AuthorizePage("/Books/Index", BookStorePermissions.Books.Default);
    options.Conventions.AuthorizePage("/Books/CreateModal", BookStorePermissions.Books.Create);
    options.Conventions.AuthorizePage("/Books/EditModal", BookStorePermissions.Books.Edit);
});
C#
Now, unauthorized users are redirected to the login page.

Hide the New Book Button
The book management page has a New Book button that should be invisible if the current user has no Book Creation permission.

bookstore-new-book-button-small

Open the file and change the content as shown below:Pages/Books/Index.cshtml

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
HTML
Added to access to the authorization service.@inject IAuthorizationService AuthorizationService
Used to check the book creation permission to conditionally render the New Book button.@if (await AuthorizationService.IsGrantedAsync(BookStorePermissions.Books.Create))
JavaScript Side
Books table in the book management page has an actions button for each row. The actions button includes Edit and Delete actions:

bookstore-edit-delete-actions

We should hide an action if the current user has not granted for the related permission. Datatables row actions has a option that can be set to to hide the action item.visiblefalse

Open the inside the project and add a option to the action as shown below:Pages/Books/Index.jsAcme.BookStore.WebvisibleEdit

{
    text: l('Edit'),
    visible: abp.auth.isGranted('BookStore.Books.Edit'), //CHECK for the PERMISSION
    action: function (data) {
        editModal.open({ id: data.record.id });
    }
}
JavaScript
Do same for the action:Delete

visible: abp.auth.isGranted('BookStore.Books.Delete')
JavaScript
abp.auth.isGranted(...) is used to check a permission that is defined before.
visible could also be get a function that returns a if the value will be calculated later, based on some conditions.bool
Menu Item
Even we have secured all the layers of the book management page, it is still visible on the main menu of the application. We should hide the menu item if the current user has no permission.

Open the class, find the code block below:BookStoreMenuContributor

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
C#
并将此代码块替换为以下代码块：

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
C#
您还需要将关键字添加到该方法并重新排列返回值。最后一堂课应如下：asyncConfigureMenuAsyncBookStoreMenuContributor

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
C#
下一部分
请参阅本教程的下一部分。

