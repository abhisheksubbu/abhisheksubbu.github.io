---
title: ASP.NET Core Web API with Swagger
layout: blog
category: [.NET]
excerpt: In this blog post, we will learn how to create an ASP.NET Core 2.2 Web API with Swagger feature. Swashbuckle is an open source project that generates swagger documents for Web API’s. Swagger makes it really easy for people to understand an API and provides a playground to interact with the Web API. Thus it ensures a rich...
comments: true
---

In this blog post, we will learn how to create an ASP.NET Core 2.2 Web API with Swagger feature. Swashbuckle is an open source project that generates swagger documents for Web API’s. Swagger makes it really easy for people to understand an API and provides a playground to interact with the Web API. Thus it ensures a rich interactivity interface for users, provides better understandable API documentation, makes it easy to discover the API and helps the developers build API’s that confirm to OpenAPI standards.

## Pre-Requisites

1. Download and Install Visual Studio 2019 Community Edition (make sure to install the feature named “ASP.NET and web development”)
2. Download and Install .NET Core 2.2 SDK

## Steps

1. Firstly, open Visual Studio 2019

   ![VS2019](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/vs2019.png)

2. Choose “**ASP.NET Core Web Application**” from project templates and click Next

   ![New Project](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/new-project.png)

3. Provide a Project Name and click “Create” button. In my case, the project name is “TestAPI”.

   ![Project Name](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/project-name.png)

4. Choose the application type as “Web Application (Model-View-Controller)“. Make sure that the .NET Core version selected is “ASP.NET Core 2.2“. Also make sure to click the “Change” link next to Authentication [on the right] and select “Individual User Accounts“. As a best practice, Web API’s needs to have authentication. We will not select Docker because it’s not in the scope of this blog. Now click the “Create” button and wait for Visual Studio to set up your project.

   ![Select Template](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/select-template.png)

5. Once Visual Studio has done setting up the project, you will see the project structure in Solution Explorer as shown below.

   ![Project Structure](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/project-structure.png)

6. Now delete the following things from the project in the Solution Explorer.

   - wwwroot (delete the entire folder)
   - Areas (delete the entire folder)
   - Models (delete the entire folder)
   - Views (delete the entire folder)
   - Controllers/HomeController.cs (delete this file only)

     ![Delete Items](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/delete-items.png)

7. Open the Startup.cs and make delete the following lines in “ConfigureServices” method
   ```csharp
   services.Configure<CookiePolicyOptions>(options =>
   {
       // This lambda determines whether user consent for non-essential cookies is needed for a given request.
       options.CheckConsentNeeded = context => true;
       options.MinimumSameSitePolicy = SameSiteMode.None;
   });
   .AddDefaultUI(UIFramework.Bootstrap4)
   ```
8. Open the Startup.cs and make delete the following lines in “Configure” method
   ```csharp
   app.UseDatabaseErrorPage();
   app.UseCookiePolicy();
   ```
9. The final code in Startup.cs should look like as shown below.

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Builder;
   using Microsoft.AspNetCore.Identity;
   using Microsoft.AspNetCore.Identity.UI;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.AspNetCore.Http;
   using Microsoft.AspNetCore.HttpsPolicy;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.EntityFrameworkCore;
   using TestAPI.Data;
   using Microsoft.Extensions.Configuration;
   using Microsoft.Extensions.DependencyInjection;

   namespace TestAPI
   {
       public class Startup
       {
           public Startup(IConfiguration configuration)
           {
               Configuration = configuration;
           }

           public IConfiguration Configuration { get; }

           // This method gets called by the runtime. Use this method to add services to the container.
           public void ConfigureServices(IServiceCollection services)
           {

               services.AddDbContext<ApplicationDbContext>(options =>
                   options.UseSqlServer(
                       Configuration.GetConnectionString("DefaultConnection")));
               services.AddDefaultIdentity<IdentityUser>()
                   .AddEntityFrameworkStores<ApplicationDbContext>();

               services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
           }

           // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
           public void Configure(IApplicationBuilder app, IHostingEnvironment env)
           {
               if (env.IsDevelopment())
               {
                   app.UseDeveloperExceptionPage();
               }
               else
               {
                   app.UseExceptionHandler("/Home/Error");
                   // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                   app.UseHsts();
               }

               app.UseHttpsRedirection();
               app.UseStaticFiles();

               app.UseAuthentication();

               app.UseMvc(routes =>
               {
                   routes.MapRoute(
                       name: "default",
                       template: "{controller=Home}/{action=Index}/{id?}");
               });
           }
       }
   }
   ```

10. Now, add an empty MVC Controller under Controllers folder and name it as “UserController”. Edit the UserController code with a GET method as shown below (this is to test if our controller would work like an API)

    - Rename the Index method to Get
    - Provide an HTTP action decoration for HTTPGET with a Url alias as “api/user”
    - Change the body of this method to return an anonymous object that has a Name property set to a value of “Test Name”

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    namespace TestAPI.Controllers
    {
        public class UserController : Controller
        {
            [HttpGet("api/user")]
            public IActionResult Get()
            {
                return Ok(new { Name = "Test Name" });
            }
        }
    }
    ```

11. Save and build the project Now, right click on the “TestAPI” project and select Properties.

    ![Project Properties](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/project-properties.png)

12. In the Project Properties, navigate to Debug tab and hit the “Delete” button next to the Profile. When you hit the Delete button, the Profile that shows “IIS Express” will change to “TestAPI” & Launch will also change to “Project”. We did this so that when we run this project, it runs as a console application. Why? Because for a Web API, you don’t have a User Interface to work with.
    ![Project Profile](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/project-profile.png)

13. Click on “Ctrl+S” to save the project properties and close this tab. Now, run this project by clicking the “F5” button on keyboard. If an SSL certificate confirmation box opens, click “Yes”.

    ![SSL Certificate](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/SSL-Certificate.png)

    ![Certificate Warning](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/certificate-warning.png)

14. You will see that a command line box opens and eventually a browser will also open. It is in this command window that the API starts and runs. The browser window opens so that you can execute the API URL and check for yourself if the API is working or not. [Do not close the command window. If you do so, the project will stop running and you have to run it again by pressing F5] On the browser window, you would see an error. This is because the API controller that we wrote is only available in the URL alias “**/api/user**“.

    ![API Run](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/api-run.png)

15. So, change the URL to “https://localhost:5001/api/user” and hit Enter. You will see the data with “name” property and “Test Name” value (exactly as we configured in the Controller code). Excellent progress till now.

    ![API JSON Response](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/api-json-response.png)

16. Now, we are ready to configure Swagger in this “TestAPI” project. Close the browser window and the command window. This stops the project from running. In the project in Visual Studio, right click on “TestAPI” project and choose “Manage NuGet Packages” option

    ![Manage Nuget](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/manage-nuget.png)

17. In the Nuget Window, change the selection from “Installed” to “Browse” and type “Swashbuckle.aspnetcore” in the Search Box. Choose the “Swashbuckle.AspNetCore” package and click on “Install”

    ![Swashbuckle](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/swashbuckle-1024x528.png)

18. If a Preview Changes window opens, click on “OK” button

    ![Preview Changes](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/preview-changes.png)

19. Once the installation of package is complete, right click on the “TestAPI” project in solution explorer –> go to “Add” –> select “New Folder“. Name the folder as “Options”.

    ![options](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/options.png)

20. Right click on “Options” folder –> go to “Add” –> select “Class”. Name the class as “SwaggerOptions.cs“. Add 3 string properties namely “JsonRoute”, “Description” and “UIEndpoint”. Save the “SwaggerOptions.cs” file by pressing Ctrl+S.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;

    namespace TestAPI.Options
    {
        public class SwaggerOptions
        {
            public string JsonRoute { get; set; }
            public string Description { get; set; }
            public string UIEndpoint { get; set; }
        }
    }
    ```

21. Open the appsettings.json file and add the SwaggerOption configurations.
    ```json
    {
      "ConnectionStrings": {
        "DefaultConnection": "Server=(localdb)mssqllocaldb;Database=aspnet-TestAPI-8323AC48-C8C7-4C5D-95E8-97EC5F60E9B8;Trusted_Connection=True;MultipleActiveResultSets=true"
      },
      "SwaggerOptions": {
        "JsonRoute": "swagger/{documentName}/swagger.json",
        "Description": "Test API",
        "UIEndpoint": "v1/swagger.json"
      },
      "Logging": {
        "LogLevel": {
          "Default": "Warning"
        }
      },
      "AllowedHosts": "*"
    }
    ```
22. Now, open the Startup.cs and add the following lines of code in “ConfigureServices” method at the last. Here, we are adding the Swagger feature to the set of services with some key information of the API.
    ```csharp
    services.AddSwaggerGen(options =>
    {
        options.SwaggerDoc("v1", new Swashbuckle.AspNetCore.Swagger.Info { Title = "Test API", Version = "v1" });
    });
    ```
23. In the “Configure” method, add the following lines of code at the last. Here, we are binding the values SwaggerOptions JSON properties and it’s values to SwaggerOptions class. Additionally, we configure the app with Swagger with the SwaggerOptions data.

    ```csharp
    //Here, we are binding the values SwaggerOptions JSON properties and it's values to SwaggerOptions class
    var swaggerOptions = new Options.SwaggerOptions();
    Configuration.GetSection(nameof(Options.SwaggerOptions)).Bind(swaggerOptions);

    //Configure the Swagger with swaggerOptions data
    app.UseSwagger(option => { option.RouteTemplate = swaggerOptions.JsonRoute; });
    app.UseSwaggerUI(option =>
    {
        option.SwaggerEndpoint(swaggerOptions.UIEndpoint, swaggerOptions.Description);
    });
    ```

24. Now, build the project. It should build without any errors. Click “F5” and run the project. In the newly opened browser, change the URL to “https://localhost:5001/swagger” and hit Enter. You should now see the Swagger UI with our “/api/user” api endpoint configured.

    ![Swagger Endpoint](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/swagger-endpoint-1024x358.png)

25. You can now test the API in Swagger UI by clicking on the “/api/user” and hit the “Try it out” button and “Execute” button. This will invoke the “/api/user” endpoint and return the response with “name” property and “Test Name” value.

    ![Execute API in Swagger](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/execute-api-in-swagger-1024x497.png)

Hope this blog was helpful to you in understanding Swagger and its configurations from scratch. Happy Coding 🙂
