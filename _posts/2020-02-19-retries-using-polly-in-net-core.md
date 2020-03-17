---
title: Retries using Polly in .NET Core
layout: blog
category: [.NET]
excerpt: In this blog, we will look at a common scenario where we have to call an API endpoint and handle retries efficiently in .NET Core. For this purpose, we will be using a library called as “Polly“.
---

In this blog, we will look at a common scenario where we have to call an API endpoint and handle retries efficiently in .NET Core. For this purpose, we will be using a library called as “Polly“.

Polly is a library that allows developers to express resilience and transient fault handling policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, and Fallback in a fluent and thread-safe manner.

### Pre-Requisites

- Open Visual Studio
- Create a New Project of type “ASP.NET Core Web Application”
- Give a good name for your project
- Choose the “MVC” template
- Hit “Create” button

### What am I trying to achieve?

The goal that I am trying to achieve is to call the [GitHub Users](https://api.github.com/users) endpoint and retrieve the data for any user in github. I want to make sure that the solution uses memory optimally & also ensures that my application retries if the API fails for some reason.

A traditional way the developers would code is to create an HttpClient, set the properties with the Uri and invoke the endpoint (as shown below).

```csharp
public async Task<IActionResult> Index()
{
    var client = new HttpClient()
    {
        BaseAddress = new Uri("https://api.github.com/"),
        DefaultRequestHeaders =
        {
            {"Accept", "application/vnd.github.v3+json" },
            {"User-Agent", "HttpClientFactoryExample" }
        }
    };
    GitHubUserModel viewModel = new GitHubUserModel();
    var userData = await client.GetStringAsync("/users/abhisheksubbu");
    viewModel.Data = JsonPrettify(userData);
    return View(viewModel);
}
```

### Problems with this traditional approach

- Every time you want the user data, the HttpClient is newly initialized
- Everytime a new HttpClient is initialized, the HTTP Handlers for the client object is also newly instantiated.
- We already know that BaseAddress and DefaultRequestHeaders are the same and do not change. So, the handlers could have been re-used but with this approach, you can’t do that.
- It would slow down the execution of this logic because all these objects have to be created in memory and then executed.

### Solution to this problem

From .NET Core 2.1 onwards, Microsoft introduced IHttpClientFactory with which we can set the handlers prior and re-use them while creating the client objects. As we all know, we can set the handlers in the Startup.cs and dependency inject it to the controller so that we can use the handlers for creating the client object.

In Startup.cs, we can specify a named HttpClient called “GitHub”

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddHttpClient("GitHub", client =>
    {
        client.BaseAddress = new Uri("https://api.github.com/");
        client.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
        client.DefaultRequestHeaders.Add("User-Agent", "HttpClientFactoryExample");
    });
}
```

In HomeController.cs, we can use httpClientFactory to create the client object using the named Client that we had specified in Startup.cs

```csharp
public class HomeController : Controller
{
    private readonly IHttpClientFactory _httpClientFactory;

    public HomeController(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<IActionResult> Index()
    {
        var client = _httpClientFactory.CreateClient("GitHub");
        GitHubUserModel viewModel = new GitHubUserModel();
        var userData = await client.GetStringAsync("/users/abhisheksubbu");
        viewModel.Data = JsonPrettify(userData);
        return View(viewModel);
    }

    private static string JsonPrettify(string json)
    {
        using (var stringReader = new StringReader(json))
        using (var stringWriter = new StringWriter())
        {
            var jsonReader = new JsonTextReader(stringReader);
            var jsonWriter = new JsonTextWriter(stringWriter) { Formatting = Formatting.Indented };
            jsonWriter.WriteToken(jsonReader);
            return stringWriter.ToString();
        }
    }
}
```

Perfect !! We now understood the optimal way to invoke API endpoints using IHttpClientFactory.

But, there can be instances where the API endpoints can fail temporarily for a fraction of a second but is likely to disappear if we invoke the API endpoint again. These kinds of temporary errors that will disappear on retry are called as “**Transient Errors**“.

Technically, Transient Errors are one of the following.

- Network failures (`System.Net.Http.HttpRequestException`)
- HTTP 5XX status codes (server errors)
- HTTP 408 status code (request timeout)
  In our case, if a transient error occurred, this call would fail naturally because there is no retry mechanism built/coded.

Microsoft provides an elegant solution to this problem with the NuGet package named “**Microsoft.Extensions.Http.Polly**“. It integrates the Polly library with IHttpClientFactory.

### Steps to implement Retry

Alright, go ahead and install the “**Microsoft.Extensions.Http.Polly**” NuGet package in the web application project.

In Startup.cs, specify the Transient Error Policy as shown below

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddHttpClient("GitHub", client =>
    {
        client.BaseAddress = new Uri("https://api.github.com/");
        client.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
        client.DefaultRequestHeaders.Add("User-Agent", "HttpClientFactoryExample");
    })
        .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, _ => TimeSpan.FromMilliseconds(300)));
}
```

Here, we are telling the IHttpClientFactory that in case of a transient error, the policy is to wait for 3 seconds (300 milliseconds) and retry the API endpoint for a maximum of 3 times. If we get the response before max retry times of 3, send the response back. If not, send the error as it is. Therefore, if this was really a transient error that would disappear on retry, essentially the user doesn’t even realize that a transient error just happened because the IHttpClientFactory immediately retried invoking the API endpoint and send back the response.

I hope this blog was useful to all .NET Core Developers. This is a common pattern that we see everyday in our coding life and you can follow the techniques mentioned here to make your applications more robust.
