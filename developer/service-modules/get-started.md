# Get Started with Phoenix Service Modules
The Phoenix platform provides a web application base for building modular web services on the ASP.Net framework. It brings together the following elements.

* ASP.Net Web API
* ASP.Net Identity Framework
* Dependency Injection using SimpleInjector

## Setting up a new web application
To begin a new web application, create a new, empty ASP.Net Web API project and install the Phoenix Web package.

```
PM> Install-Package Phoenix.Web
```

Then add a new `Startup.cs` file with the following contents.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using Microsoft.Owin;
using Owin;
using Phoenix.Web;

[assembly: OwinStartup(typeof(MyWebApplication.Startup))]

namespace MyWebApplication
{
    public class Startup
    {

        public void Configuration(IAppBuilder app)
        {
            app.UsePhoenix();
        }
    }
}
```

That's all you have to do to get a modular web application running using the Phoenix platform.

## Add a new module
To create a new service module, add a new empty class library project to your solution.

It is recommended that you add a folder for `Controllers` and `Permissions` but you may put these classes where ever you like.

![Module Structure](resources/get-started-solution.png)

Add a class to define information about your module. For this example, we've added  the `MyModule` class which implements the `IPhoenixModule` interface. Implement the required members as follows.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MyModule
{
    public class MyModule : IPhoenixModule
    {
        public string Name
        {
            get
            {
                return "My Module";
            }
        }

        public string Description
        {
            get
            {
                return "My Module does some really great stuff.";
            }
        }

        /// <summary>
        /// Returns all permission groups for the module
        /// </summary>
        public IEnumerable<IPermissionGroup> PermissionGroups
        {
            get
            {
                return new List<IPermissionGroup>()
                {
                    new Permissions.MyFeaturePermissions()
                };
            }
        }

        public MyModule(/*Use dependency injection to inject any dependencies here*/)
        {
            //Module initialisation code goes here.
        }

        public void Configure(params object[] args)
        {
            //You can configure the module here.
            //This is called when the web application starts.
        }
    }
}

```

You may have noticed the `PermissionGroups` member. We'll get to that in the next section. For now, you may simply add Web API controllers to the controllers folder and compile. Copy the compiled dll to the `bin/Modules` under the MyWebApplication project. The dll is loaded and everything is ready for you to start using the controllers in this module.

### Configuring module permissions
You can configure module permissions for use with the `Phoenix.Security` package.

Install the `Phoenix.Security` package using the following command.

```
PM> Install-Package Phoenix.Security
```

Service modules, expose a list of `PermissionGroups`. A `PermissionGroup` is intended to define a set of permissions for a specific feature of your module. For example, if your module deals with products and you want to secure these you might define a permission group for products with the following permissions.

* Can View Product
* Can Create Product
* Can Edit Product
* Can Delete Product

This will allow you to control all CRUD related aspects of a product where only certain users can delete a product.

A permission group can be define as follows.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MyModule.Permissions
{
    public class MyFeaturePermissions
    {
        public string Description
        {
            get
            {
                return "Controls access to process my feautre.";
            }
        }

        public string Name
        {
            get
            {
                return "My Feature";
            }
        }

        public IEnumerable<IPermission> Permissions
        {
            get
            {
                return new List<IPermission>
                {
                    new CanViewMyFeature(),
                    new CanCreateMyFeature(),
                    new CanDeleteMyFeature(),
                    new CanEditMyFeature()
                };
            }
        }

        public class CanViewMyFeature : PermissionBase, IPermission
        {
            public string Name { get { return "View"; } }
            public string Description { get { return "Provides access to view My Feature"; } }
        }

        public class CanCreateMyFeature : PermissionBase, IPermission
        {
            public string Name { get { return "Create"; } }
            public string Description { get { return "Provides access to create new My Feature"; } }
        }

        public class CanDeleteMyFeature : PermissionBase, IPermission
        {
            public string Name { get { return "Delete"; } }
            public string Description { get { return "Provides access to delete My Feature"; } }
        }

        public class CanEditMyFeature : PermissionBase, IPermission
        {
            public string Name { get { return "Edit"; } }
            public string Description { get { return "Provides access to change My Feature"; } }
        }
    }
}
```

To use permissions in a controller or anywhere throughout your code, you can use the `IPhoenixAccess` component can be acquired using dependency injection. The following code demonstrates this.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Web.Http;
using Phoenix.Access;
using static MyModule.Permissions.MyFeaturePermissions;

namespace MyModule.Controllers
{
    public class MyFeatureController
    {
        IPhoenixAccess access;

        public MyFeatureController(IPhoenixAccess access /*Injected via DI*/)
        {
            this.access = access;
        }

        /// <summary>
        /// List all existing items of my feature
        /// </summary>
        /// <returns>My features items</returns>
        [HttpGet]
        [Route("")]
        public IEnumerable<string> ListItems()
        {
            access.CheckAccess<CanViewMyFeature>(RequestContext.Principal);

            return new[] { "Item 1", "Item 2" };
        }
    }
}

```

If the current user (Principal) does not have access for this permission an exception will be thrown and the code will not continue to execute.

## Adding an existing module
You can use NuGet to install a module from a NuGet feed.

Comming Soon!