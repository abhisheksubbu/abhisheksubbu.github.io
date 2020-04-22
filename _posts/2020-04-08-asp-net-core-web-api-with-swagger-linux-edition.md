---
title: ASP.NET Core Web API with Swagger [Linux Edition]
layout: blog
category: [.NET, Deep Dive]
excerpt: In this blog post, we will learn how to create an ASP.NET Core Web API with Swagger feature in a Linux environment. Swashbuckle is an open source project that generates swagger documents for Web API’s. Swagger makes it really easy for people to understand an API and provides a playground to interact with the Web API. Thus it ensures a rich...
comments: true
---

In this blog post, we will learn how to create an ASP.NET Core Web API with Swagger feature in a Linux environment. Swashbuckle is an open source project that generates swagger documents for Web API’s. Swagger makes it really easy for people to understand an API and provides a playground to interact with the Web API. Thus it ensures a rich interactivity interface for users, provides better understandable API documentation, makes it easy to discover the API and helps the developers build API’s that confirm to OpenAPI standards.

> If you want to read the Swagger setup in Visual Studio Community Edition in Windows OS, [click here]({{site.baseurl}}/asp-net-core-2-2-web-api-with-swagger/){:target="\_blank"}

## Pre-Requisites

1. You should be running Ubuntu Linux
1. Download and Install Visual Studio Code
1. Download and Install the latest .NET Core SDK

## Steps

1. Open Terminal and create an ASP.NET MVC .net core project with "Individual" authentication
   ![Terminal Command](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/dotnet-new-mvc-swagger.png)

1. Open Visual Studio Code from the project location. You will see the project structure in Explorer as shown below.

   ![Project Structure](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/vscode-project-explorer.png)

1. Now delete the following things from the project in the Solution Explorer.

   - wwwroot (delete the entire folder)
   - Areas (delete the entire folder)
   - Models (delete the entire folder)
   - Views (delete the entire folder)
   - Controllers/HomeController.cs (delete this file only)

1. Open the Startup.cs and make delete the following lines in “Configure” method
   ```csharp
   app.UseDatabaseErrorPage();
   ```
1. The final code in Startup.cs should look like as shown below.

   ```csharp
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Identity;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.EntityFrameworkCore;
    using asp_net_core_swagger.Data;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Hosting;

    namespace asp_net_core_swagger
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
                    options.UseSqlite(
                        Configuration.GetConnectionString("DefaultConnection")));
                services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
                    .AddEntityFrameworkStores<ApplicationDbContext>();
                services.AddControllersWithViews();
            services.AddRazorPages();
            }

            // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
            public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
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

                app.UseRouting();

                app.UseAuthentication();
                app.UseAuthorization();

                app.UseEndpoints(endpoints =>
                {
                    endpoints.MapControllerRoute(
                        name: "default",
                        pattern: "{controller=Home}/{action=Index}/{id?}");
                    endpoints.MapRazorPages();
                });
            }
        }
    }
   ```

1. Now, add an empty MVC Controller under Controllers folder and name it as “UserController”. Edit the UserController code with a GET method as shown below (this is to test if our controller would work like an API)

   - Rename the Index method to Get
   - Provide an HTTP action decoration for HTTPGET with a Url alias as “api/user”
   - Change the body of this method to return an anonymous object that has a Name property set to a value of “Test Name”

   ```csharp
    using Microsoft.AspNetCore.Mvc;

    namespace asp_net_core_swagger
    {
        public class UserController : Controller
        {
            [HttpGet("api/user")]
            public IActionResult Get()
            {
                return Ok(new {Name = "Test Name"});
            }
        }
    }
   ```

1. Click Ctrl+F5 to build and run the project.

1. On the browser window, you would see an error. This is because the API controller that we wrote is only available in the URL alias “**/api/user**“.

   ![API Run](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/api-run.png)

1. So, change the URL to “https://localhost:5001/api/user” and hit Enter. You will see the data with “name” property and “Test Name” value (exactly as we configured in the Controller code). Excellent progress till now.

   ![API JSON Response](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/api-json-response.png)

1. Now, we are ready to configure Swagger in the project. Close the browser window. This stops the project from running. In Visual Studio Code, click Ctrl+` to open the Terminal window inside vscode.

   ![Manage Nuget](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/vscode-terminal.png)

1. In the Terminal Window,type the following command to install the "Swashbuckle.AspNetCore" nuget package.

   ```console
   dotnet add package Swashbuckle.AspNetCore --version 5.3.1
   ```

1. Once the installation of package is complete, right click on the project in the explorer –> create a new folder named as “Options”.

   ![options](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/swaggerOptions-file.png)

1. Right click on “Options” folder –> go to “Add” –> select “Class”. Name the class as “SwaggerOptions.cs“. Add 3 string properties namely “JsonRoute”, “Description” and “UIEndpoint”. Save the “SwaggerOptions.cs” file by pressing Ctrl+S.

   ```csharp
    namespace asp_net_core_swagger.Options
    {
        public class SwaggerOptions
        {
            public string JsonRoute { get; set; }
            public string Description { get; set; }
            public string UIEndpoint { get; set; }
        }
    }
   ```

1. Open the appsettings.json file and add the SwaggerOption configurations.
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
1. Now, open the Startup.cs and add the following lines of code in “ConfigureServices” method at the last. Here, we are adding the Swagger feature to the set of services with some key information of the API.
   ```csharp
   services.AddSwaggerGen(options =>
   {
       options.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo { Title = "Test API", Version = "v1" });
   });
   ```
1. In the “Configure” method, add the following lines of code at the last. Here, we are binding the values SwaggerOptions JSON properties and it’s values to SwaggerOptions class. Additionally, we configure the app with Swagger with the SwaggerOptions data.

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

1. Now, build the project. It should build without any errors. Click “F5” and run the project. In the newly opened browser, change the URL to “https://localhost:5001/swagger” and hit Enter. You should now see the Swagger UI with our “/api/user” api endpoint configured.

   ![Swagger Endpoint](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/swagger-endpoint-1024x358.png)

1. You can now test the API in Swagger UI by clicking on the “/api/user” and hit the “Try it out” button and “Execute” button. This will invoke the “/api/user” endpoint and return the response with “name” property and “Test Name” value.

   ![Execute API in Swagger](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/execute-api-in-swagger-1024x497.png)

Hope this blog was helpful to you in understanding Swagger and its configurations from scratch. Happy Coding 🙂
