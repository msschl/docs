---
title: Use HttpClientFactory to implement resilient HTTP requests
description: Learn how to use HttpClientFactory, available since .NET Core 2.1, for creating `HttpClient` instances, making it easy for you to use it in your applications. 
ms.date: 08/08/2019
---
# Use HttpClientFactory to implement resilient HTTP requests

`HttpClientFactory` is an opinionated factory, available since .NET Core 2.1, for creating <xref:System.Net.Http.HttpClient> instances to be used in your applications.

## Issues with the original HttpClient class available in .NET Core

The original and well-known <xref:System.Net.Http.HttpClient> class can be easily used, but in some cases, it isn't being properly used by many developers.

As a first issue, while this class is disposable, using it with the `using` statement is not the best choice because even when you dispose `HttpClient` object, the underlying socket is not immediately released and can cause a serious issue named ‘sockets exhaustion’. For more information about this issue, see [You're using HttpClient wrong and it's destabilizing your software](https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/) blog post.

Therefore, `HttpClient` is intended to be instantiated once and reused throughout the life of an application. Instantiating an `HttpClient` class for every request will exhaust the number of sockets available under heavy loads. That issue will result in `SocketException` errors. Possible approaches to solve that problem are based on the creation of the `HttpClient` object as singleton or static, as explained in this [Microsoft article on HttpClient usage](../../../csharp/tutorials/console-webapiclient.md).

But there’s a second issue with `HttpClient` that you can have when you use it as singleton or static object. In this case, a singleton or static `HttpClient` doesn't respect DNS changes, as explained in this [issue](https://github.com/dotnet/corefx/issues/11224) at the dotnet/corefx GitHub repository.

To address those mentioned issues and make the management of `HttpClient` instances easier, .NET Core 2.1 introduced a new `HttpClientFactory` that can also be used to implement resilient HTTP calls by integrating Polly with it.

[Polly](http://www.thepollyproject.org/) is transient-fault-handling library that helps developers add resiliency to their applications, by using some pre-defined policies in a fluent and thread-safe manner.

## What is HttpClientFactory

`HttpClientFactory` is designed to:

- Provide a central location for naming and configuring logical `HttpClient` objects. For example, you may configure a client (Service Agent) that's pre-configured to access a specific microservice.
- Codify the concept of outgoing middleware via delegating handlers in `HttpClient` and implementing Polly-based middleware to take advantage of Polly’s policies for resiliency.
- `HttpClient` already has the concept of delegating handlers that could be linked together for outgoing HTTP requests. You register HTTP clients into the factory and you can use a Polly handler to use Polly policies for Retry, CircuitBreakers, and so on.
- Manage the lifetime of `HttpClientMessageHandlers` to avoid the mentioned problems/issues that can occur when managing `HttpClient` lifetimes yourself.

> [!NOTE]
> `HttpClientFactory` is tightly tied to the dependency injection (DI) implementation in the `Microsoft.Extensions.DependencyInjection` NuGet package. For more information about using other dependency injection containers, see this [GitHub discussion](https://github.com/aspnet/Extensions/issues/1345).

## Multiple ways to use HttpClientFactory

There are several ways that you can use `HttpClientFactory` in your application:

- Use `HttpClientFactory` Directly
- Use Named Clients
- Use Typed Clients
- Use Generated Clients

For the sake of brevity, this guidance shows the most structured way to use `HttpClientFactory`, which is to use Typed Clients (Service Agent pattern). However, all options are documented and are currently listed in this [article covering HttpClientFactory usage](/aspnet/core/fundamentals/http-requests#consumption-patterns).

## How to use Typed Clients with HttpClientFactory

So, what's a "Typed Client"? It's just an `HttpClient` that's configured upon injection by the `DefaultHttpClientFactory`.

The following diagram shows how Typed Clients are used with `HttpClientFactory`:

![Diagram showing how typed clients are used with HttpClientFactory.](./media/use-httpclientfactory-to-implement-resilient-http-requests/client-application-code.png)

**Figure 8-4**. Using HttpClientFactory with Typed Client classes.

In the above image, a ClientService (used by a controller or client code) uses an `HttpClient` created by the registered `IHttpClientFactory`. This factory assigns the `HttpClient` an `HttpMessageHandler` from a pool it manages. The `HttpClient` can be configured with Polly's policies when registering the `IHttpClientFactory` in the DI container with the extension method `AddHttpClient`.

To configure the above structure, add `HttpClientFactory` in your application by installing the `Microsoft.Extensions.Http` NuGet package that includes the `AddHttpClient()` extension method for `IServiceCollection`. This extension method registers the `DefaultHttpClientFactory` to be used as a singleton for the interface `IHttpClientFactory`. It defines a transient configuration for the `HttpMessageHandlerBuilder`. This message handler (`HttpMessageHandler` object), taken from a pool, is used by the `HttpClient` returned from the factory.

In the next code, you can see how `AddHttpClient()` can be used to register Typed Clients (Service Agents) that need to use `HttpClient`.

```csharp
// Startup.cs
//Add http client services at ConfigureServices(IServiceCollection services)
services.AddHttpClient<ICatalogService, CatalogService>();
services.AddHttpClient<IBasketService, BasketService>();
services.AddHttpClient<IOrderingService, OrderingService>();
```

Registering the client services as shown in the previous code, makes the `DefaultClientFactory` create a standard `HttpClient` for each service.

You could also add instance-specific configuration in the registration to, for example, configure the base address, and add some resiliency policies, as shown in the following code:

```csharp
services.AddHttpClient<ICatalogService, CatalogService>(client =>
{
    client.BaseAddress = new Uri(Configuration["BaseUrl"]);
})
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());
```

Just for the example sake, you can see one of the above policies in the next code:

```csharp
static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .OrResult(msg => msg.StatusCode == System.Net.HttpStatusCode.NotFound)
        .WaitAndRetryAsync(6, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
}
```

You can find more details about using Polly in the [Next article](implement-http-call-retries-exponential-backoff-polly.md).

### HttpClient lifetimes

Each time you get an `HttpClient` object from the `IHttpClientFactory`, a new instance is returned. But each `HttpClient` uses an `HttpMessageHandler` that's pooled and reused by the `IHttpClientFactory` to reduce resource consumption, as long as the `HttpMessageHandler`'s lifetime hasn't expired.

Pooling of handlers is desirable as each handler typically manages its own underlying HTTP connections; creating more handlers than necessary can result in connection delays. Some handlers also keep connections open indefinitely, which can prevent the handler from reacting to DNS changes.

The `HttpMessageHandler` objects in the pool have a lifetime that's the length of time that an `HttpMessageHandler` instance in the pool can be reused. The default value is two minutes, but it can be overridden per Typed Client. To override it, call `SetHandlerLifetime()` on the `IHttpClientBuilder` that's returned when creating the client, as shown in the following code:

```csharp
//Set 5 min as the lifetime for the HttpMessageHandler objects in the pool used for the Catalog Typed Client
services.AddHttpClient<ICatalogService, CatalogService>()
    .SetHandlerLifetime(TimeSpan.FromMinutes(5));
```

Each Typed Client can have its own configured handler lifetime value. Set the lifetime to `InfiniteTimeSpan` to disable handler expiry.

### Implement your Typed Client classes that use the injected and configured HttpClient

As a previous step, you need to have your Typed Client classes defined, such as the classes in the sample code, like ‘BasketService’, ‘CatalogService’, ‘OrderingService’, etc. – A Typed Client is a class that accepts an `HttpClient` object (injected through its constructor) and uses it to call some remote HTTP service. For example:

```csharp
public class CatalogService : ICatalogService
{
    private readonly HttpClient _httpClient;
    private readonly string _remoteServiceBaseUrl;

    public CatalogService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<Catalog> GetCatalogItems(int page, int take,
                                               int? brand, int? type)
    {
        var uri = API.Catalog.GetAllCatalogItems(_remoteServiceBaseUrl,
                                                 page, take, brand, type);

        var responseString = await _httpClient.GetStringAsync(uri);

        var catalog = JsonConvert.DeserializeObject<Catalog>(responseString);
        return catalog;
    }
}
```

The Typed Client (CatalogService in the example) is activated by DI (Dependency Injection), meaning that it can accept any registered service in its constructor, in addition to HttpClient.

A Typed Client is, effectively, a transient object, meaning that a new instance is created each time one is needed and it will receive a new `HttpClient` instance each time it's constructed. However, the HttpMessageHandler objects in the pool are the objects that are reused by multiple Http requests.

### Use your Typed Client classes

Finally, once you have your typed classes implemented and have them registered with `AddHttpClient()`, you can use them wherever you can have services injected by DI. For example, in a Razor page code or controller of an MVC web app, like in the following code from eShopOnContainers:

```csharp
namespace Microsoft.eShopOnContainers.WebMVC.Controllers
{
    public class CatalogController : Controller
    {
        private ICatalogService _catalogSvc;

        public CatalogController(ICatalogService catalogSvc) =>
                                                           _catalogSvc = catalogSvc;

        public async Task<IActionResult> Index(int? BrandFilterApplied,
                                               int? TypesFilterApplied,
                                               int? page,
                                               [FromQuery]string errorMsg)
        {
            var itemsPage = 10;
            var catalog = await _catalogSvc.GetCatalogItems(page ?? 0,
                                                            itemsPage,
                                                            BrandFilterApplied,
                                                            TypesFilterApplied);
            //… Additional code
        }

        }
}
```

Up to this point, the code shown is just performing regular Http requests, but the ‘magic’ comes in the following sections where, just by adding policies and delegating handlers to your registered Typed Clients, all the HTTP requests to be done by `HttpClient` will behave taking into account resilient policies such as retries with exponential backoff, circuit breakers, or any other custom delegating handler to implement additional security features, like using auth tokens, or any other custom feature.

## Additional resources

- **Using HttpClientFactory in .NET Core**  
  [https://docs.microsoft.com/aspnet/core/fundamentals/http-requests](/aspnet/core/fundamentals/http-requests)

- **HttpClientFactory source code in the `aspnet/Extensions` GitHub repository**  
  <https://github.com/aspnet/Extensions/tree/master/src/HttpClientFactory>

- **Polly (.NET resilience and transient-fault-handling library)**  
  <http://www.thepollyproject.org/>
  
- **Using HttpClientFactory without dependency injection (GitHub issue)**  
  <https://github.com/aspnet/Extensions/issues/1345>

>[!div class="step-by-step"]
>[Previous](explore-custom-http-call-retries-exponential-backoff.md)
>[Next](implement-http-call-retries-exponential-backoff-polly.md)
