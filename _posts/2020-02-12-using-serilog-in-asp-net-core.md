---
title: Using Serilog in ASP.NET Core
layout: blog
category: [.NET]
excerpt: Serilog is a logging implementation that can be plugged into ASP.NET Core. It supports structured logging API’s and receives log events from the ASP.NET Core framework class libraries. In this blog we will see how to configure Serilog in ASP.Net Core project.
comments: true
---

[Serilog](https://serilog.net/) is a logging implementation that can be plugged into ASP.NET Core. It supports structured logging API’s and receives log events from the ASP.NET Core framework class libraries. In this blog we will see how to configure Serilog in ASP.Net Core project.

1. Firstly, create a fresh ASP.NET Core MVC application.
2. Add the “**Serilog.AspNetCore**” Nuget package into the project.
3. If needed, you can add top-level try-catch block in Program.cs (if you intend to log top level application logs related to host)

   - In the Main() method, we are creating the Serilog’s Log File
     - Specify the default log level [which is DEBUG in this case (line no.15)]
     - Override logs from certain sources to avoid noise in the generated logs [line no.15 and 16]
     - Enrich the log object with data from the HttpContext
     - Specify where the log has to be written (like Console, Seq, etc)
   - In the CreateHostBuilder method, we are connecting the Serilog to the HostBuilder.

   ```csharp
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.Extensions.Hosting;
   using Serilog;
   using Serilog.Events;
   using System;

   namespace SerilogTest
   {
       public class Program
       {
           public static int Main(string[] args)
           {
               //CreateHostBuilder(args).Build().Run();
               Log.Logger = new LoggerConfiguration()
               .MinimumLevel.Debug()
               .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
               .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning)
               .Enrich.FromLogContext()
               .WriteTo.Console()
               .WriteTo.Seq("http://localhost:5341")
               .CreateLogger();

               // Wrap creating and running the host in a try-catch block
               try
               {
                   Log.Information("Starting host");
                   CreateHostBuilder(args).Build().Run();
                   return 0;
               }
               catch (Exception ex)
               {
                   Log.Fatal(ex, "Host terminated unexpectedly");
                   return 1;
               }
               finally
               {
                   Log.CloseAndFlush();
               }
           }

           public static IHostBuilder CreateHostBuilder(string[] args) =>
               Host.CreateDefaultBuilder(args)
                   .UseSerilog()
                   .ConfigureWebHostDefaults(webBuilder =>
                   {
                       webBuilder.UseStartup<Startup>();
                   });
       }
   }
   ```

   > You can remove framework event noise (which you are probably not interested) by adding MinimumLevel.Override [as written in line no.16 and 17 of Program.cs]

4. In startup.cs, add the app.UseSerilogRequestLogging(); so that serilog middleware gets connected to the request pipeline.

   ```csharp
   using Microsoft.AspNetCore.Builder;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.Extensions.Configuration;
   using Microsoft.Extensions.DependencyInjection;
   using Microsoft.Extensions.Hosting;
   using Serilog;

   namespace SerilogTest
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
                   app.UseExceptionHandler("/Error");
                   // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                   app.UseHsts();
               }

               app.UseHttpsRedirection();
               app.UseStaticFiles();

               app.UseSerilogRequestLogging();

               app.UseRouting();

               app.UseAuthorization();

               app.UseEndpoints(endpoints =>
               {
                   endpoints.MapRazorPages();
               });
           }
       }
   }
   ```

5. Run the application and see the logs in the Output window. The highlighted information is generated by Serilog. The information log shows the following:
   1. Protocol
   2. Action Verb
   3. URL
   4. HTTP Status Code
   5. Time Elapsed

![Output Window](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/serilog-information.png)
