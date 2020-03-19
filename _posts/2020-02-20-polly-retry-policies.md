---
title: Polly Retry Policies
layout: blog
category: [.NET]
excerpt: If you have followed my blog on implementing “Retries using Polly in .NET Core”, then this blog will make more sense to you. In this blog, we will understand how many different techniques of Retry policies can be used in Polly.
comments: true
---

If you have followed my blog on implementing “Retries using Polly in .NET Core”, then this blog will make more sense to you. In this blog, we will understand how many different techniques of Retry policies can be used in Polly.

There are 4 different ways in which Transient Error Policy can handle faults.

1. Retry
2. Retry Forever
3. Wait and Retry
4. Wait and Retry Forever
5. Circuit Breaker
6. Advanced Circuit Breaker
7. Fallback

## 1. Retry

Retry simply dictates the handler to retry when a transient error occurs.

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
        .AddTransientHttpErrorPolicy(policy => policy.RetryAsync()); //Retries only once
                .AddTransientHttpErrorPolicy(policy => policy.RetryAsync(3)); //Retries only for 3 times
                .AddTransientHttpErrorPolicy(policy => policy.RetryAsync(3, onRetry: (exception, retryCount)=>
                {
                    //Add logic to be executed before each retry
                }));//Retries only for 3 times but on every retry, it can execute the logic specified
}
```

## 2. Retry Forever

Retry Forever means that the handler keeps retrying until it succeeds. So, here there is no concept of retry count.

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
        .AddTransientHttpErrorPolicy(policy => policy.RetryForeverAsync()); //Retries until it succeeds
                .AddTransientHttpErrorPolicy(policy => policy.RetryForeverAsync(onRetry: (exception)=>
                {
                    //Add logic to be executed before each retry
                }));//Retries forever but on every retry, it can execute the logic specified
}
```

## 3. Wait and Retry

Wait and Retry means that the handler first waits for the the specified amount of time and then retries. Here, you can specify a max retry count so that the handler can give up if it doesn’t get a response in the max retries.

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
        .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, _ => TimeSpan.FromMilliSeconds(500)); //Wait for 5 seconds and then Retries for a maximum of 3 times
                .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, _ => TimeSpan.FromMilliseconds(500), (exception, timeSpan) =>
                {
                    //Add logic to be executed before each retry
                })); //Wait for 5 seconds, execute some logic and then Retry for a maximum of 3 times.
}
```

## 4. Wait and Retry Forever

Wait and Retry Forever means that the handler first waits for the the specified amount of time and then retries until it succeeds. Here, there is no concept of retry count because it is a forever technique.

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
        .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryForeverAsync(_ => TimeSpan.FromMilliSeconds(500)); //Wait for 5 seconds and then Retries until it succeeds
                .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryForeverAsync(_ => TimeSpan.FromMilliseconds(500), (exception, timeSpan) =>
                {
                    //Add logic to be executed before each retry
                })); //Wait for 5 seconds, execute some logic and then Retries until it succeeds
}
```

## 5. Circuit Breaker

Circuit Breaker is a technique in which the entire operations are stopped/allowed with respect the conditions that we specify in the circuit breaker. It’s very similar to the Miniature Circuit Breaker (MCB) electrical component that we use at our homes to protect the house from power surge.

![Miniature Circuit Breaker (MCB) used for home](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/home-electrical-mcb.jpg)

A circuit breaker (from .NET core – Polly perspective) can put the transient error handler in 4 states.

- Open – Normal operation. Execution of actions allowed.
- Closed – The automated controller has opened the circuit. Execution of actions blocked.
- HalfOpen – Recovering from open state, after the automated break duration has expired. Execution of actions permitted. Success of subsequent action/s controls onward transition to Open or Closed state.
- Isloated – Circuit held manually in an open state. Execution of actions blocked.

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
        .AddTransientHttpErrorPolicy(policy => policy.CircuitBreakerAsync(2, TimeSpan.FromMinutes(1))); //Stops the handler (Circuit State to Open) for 1 minute if exceptions are encountered for 2 consecutive times. After 1 minute, the handler starts operating normally (Circuit State to Closed).
}
```

## 6. Advanced Circuit Breaker

Advanced Circuit Breaker helps us to measure the amount of transient faults occurring in the system and break the circuit accordingly and reset back after a particular duration.

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
        .AddTransientHttpErrorPolicy(policy => policy.AdvancedCircuitBreakerAsync(
                    failureThreshold:0.5,
                    samplingDuration: TimeSpan.FromSeconds(10),
                    minimumThroughput: 8,
                    durationOfBreak: TimeSpan.FromSeconds(30)
                    ));
                // Breaks the circuit for 30 seconds if 50% of above of incoming requests fails OR a minimum of 8 faults occur within a 10 second period. The circuit is reset/closed after 30 seconds.
}
```

## 7. Fallback

Fallback technique helps to return a fallback value instead of exception re-throw when faults keep occurring. This helps the system to ensure that it gracefully tries to keep the system stable when it detects a fallback value coming by.

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
        .AddTransientHttpErrorPolicy(policy => policy.FallbackAsync(new System.Net.Http.HttpResponseMessage(System.Net.HttpStatusCode.RequestTimeout)));
                // Returns a fallback value
}
```
