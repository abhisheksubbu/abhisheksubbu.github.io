---
title: API Versioning
layout: blog
category: [.NET, Deep Dive]
excerpt: In this blog post, we will closely look at different strategies to versioning API's
comments: true
---

## Why should we version our API's?

The first time we publish an API for clients to consume, then the users expect that the API's do not change suddenly and break their integrations. But the fact is that new requirements will come to the developers maintaining the API and they have to somehow evolve the API's without breaking the client integrations at any cost.

So, the only way around this is to version the API's as new changes are rolled out. This gives clients the flexibility to use the previous versions of API and continue their business. They can parallely test the new API version on their sandbox and once tested successfully, roll over to the new API version.

## Typical scenarios where API versioning can cause problems

- If the previous version of the API targets a set of assemblies & the newer version of the same API targets a different/upgraded version of the assemblies, then it becomes difficult to deploy both sets of assemblies side by side in the same API codebase.

## Approaches to API versioning

> Note that there is no one right way to achieve this. The approach typically depends on your requirements, the way you anticipate changes to occur in your API and the way your clients find it easy to incorporate API verioning changes. So, don't complicate API versioning with a single best practice. The fact is that there is no single solution to versioning.

### Approach 1 : URI Versioning

This approach versions API endpoints based on the URI. For example, `https://abc.org/api/v2/customers`. Personally, I don't like this approach as the root URI itself changes and everytime clients consuming the API have to change the API URI's.

### Approach 2 : Query String Versioning

This approach versions API endpoints based on the query string parameter. For example, `https://abc.org/api/customers?v=2.0`. This is slightly better than the URI approach since the root API URI doesn't change but query string parameter still kind of pollutes the URI.

### Approach 3 : Request Header Versioning

This approach versions API endpoints based on the `X-Version` header. For example,

```
GET /api/customers HTTP/1.1
Host: localhost:4000
Content-Type: application/json
X-Version: 2.0
```

In this approach, the good thing is that the API URI is consistent throughout and looks beautiful. But your clients need to know how to generate requests by setting the `X-Version` header value to the version that they want to integrate into. This is better than the Approach 1 and 2.

### Approach 4 : Accept Header Versioning

This approach versions API endpoints based on the `Accept` header. For example,

```
GET /api/customers HTTP/1.1
Host: localhost:4000
Content-Type: application/json
Accept: application/json;version=2.0
```

In this approach, the good thing is that the API URI is consistent throughout and looks beautiful. Another advantage of this approach is that we can set the Accept header value when we are setting the data type to be passed into the API endpoint. This is more convenient but slightly more complicated than Approach 3. Remember that the structure of the content that a version accepts may be different from the structure that the later version accepts.

### Approach 5 : Content Type Versioning

This approach versions API endpoints based on the `Content-Type` header. For example,

```
GET /api/customers HTTP/1.1
Host: localhost:4000
Content-Type: application/vnd.yourapp.customer.v2+json
Accept: application/vnd.yourapp.customer.v2+json
```

This approach gets a little more complex but it allows you to specify what version of the content is being send and what version of the content can be received. It is typically useful in scenarios where you are sending data to an API that got some granular changes. So, you can specify the version of the content that you are sending in `Accept` header and you can talk to the specific version of the API using the `Content-Type` header. This approach is more bullet proof as it lets you play with `Content-Type` and `Accept` value combinations. But understand that this is comparitively harder to accomplish.

> Personally, I would stick to Approach 3 or Approach 2 as it is more easier for clients to consume it. Unless your API versioning needs are too stringent, you could definitely go for Approach 4 or 5 considering that the challenges & complexities that come along with it.

Let's see now dive into an ASP.Net Core Web API setup and configure the API versions **[this is specifically the Approach 3]**

1. Create a folder named 'asp-net-core-version'
2. Inside this folder, create a webapi project using .NET CLI command

   ```console
       dotnet new webapi
   ```

3. Add the AspNet Core Versioning Nuget Package

   ```console
       dotnet add package Microsoft.AspNetCore.Mvc.Versioning
   ```

4. Configure the ApiVersioning middleware as follows

   ```csharp
       services.AddApiVersioning(config => {
               config.DefaultApiVersion = new ApiVersion(1,0); //we are telling the middleware to assume 1.0 as the default API version
               config.AssumeDefaultVersionWhenUnspecified = true; //if the consumer doesn't specify API version, we are instructing the use the default api version
               config.ReportApiVersions = true; // this reports the supported api version numbers back in the api response to the client. This way clients know what api versions to use.
               config.ApiVersionReader = new HeaderApiVersionReader("X-Version"); // this instructs the middleware to detect the API version only from the request header 'X-Version'
           });
   ```

5. In `Configure()` of the Startup.cs, use the ApiVersioning middleware
   ```csharp
       app.UseApiVersioning();
   ```
6. I created 2 versions of CustomerController and wrote the `GET()`. Notice the MapTpApiVerion attribute tells the versioning middleware to which controller should it route to.

   ```csharp
    using System.Collections.Generic;
    using Microsoft.AspNetCore.Mvc;
    using asp_net_core_version;

    namespace asp_net_core_version.Controllers
    {
        [ApiController]
        [ApiVersion("1.0")]
        [Route("api/customer")]
        public class CustomerController: ControllerBase
        {
            [HttpGet]
            public IActionResult GetCustomers()
            {
                var customers = new List<Models.v10.Customer> {
                    new Models.v10.Customer { Name = "Abhishek" },
                    new Models.v10.Customer { Name = "John" }
                };
                return Ok(customers);
            }
        }

        [ApiController]
        [ApiVersion("1.1")]
        [Route("api/customer")]
        public class Customer2Controller: ControllerBase
        {
            [HttpGet]
            public IActionResult GetCustomers()
            {
                var customers = new List<Models.v11.Customer> {
                    new Models.v11.Customer { Name = "Abhishek", Age = 32 },
                    new Models.v11.Customer { Name = "John", Age = 35 }
                };

                return Ok(customers);
            }
        }
    }
   ```

7. Also note that I have a folder named "Models" inside which there are folders specific to versions like "v10", "v11" in which I separate out the models

   ```csharp
   using System;
   namespace asp_net_core_version.Models.v10
   {
       public class Customer
       {
           public string Name { get; set; }
       }
   }

   namespace asp_net_core_version.Models.v11
   {
       public class Customer
       {
           public string Name { get; set; }
           public int Age { get; set; }
       }
   }

   ```

8. Now, build and run the API project.
9. Open PostMan and test the `/api/customers` endpoint without any header value. You should get the following response.
   ```json
   [
     {
       "name": "Abhishek"
     },
     {
       "name": "John"
     }
   ]
   ```
10. Open PostMan and test the `/api/customers` endpoint with header key as **X-Version** and value set as **1.1**. You should get the following response.
    ```json
    [
      {
        "name": "Abhishek",
        "age": 32
      },
      {
        "name": "John",
        "age": 35
      }
    ]
    ```

Also notice that every API response contains a header key called as "**api-supported-versions**" with values "**1.0, 1.1**". This tells the consumer of the API that you can set the `X-Version` to either 1.0 or 1.1 since the API allows these versions.

![Api Version Response](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/api-versions-response.png)
