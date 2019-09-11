# ASP.NET Core 3.0 and Blazor

This file/repository is aimed to group all the breaking changes and new features that got introduced in version 3.0 for a quick read/search.

## Table of content

- [ASP.NET Core 3.0 and Blazor](#aspnet-core-30-and-blazor)
  - [Table of content](#table-of-content)
  - [ASP.NET Core](#aspnet-core)
    - [ASP.NET Core 3.0 will only run on .NET Core](#aspnet-core-30-will-only-run-on-net-core)
    - [`Microsoft.AspNetCore.All` has been deprecated](#microsoftaspnetcoreall-has-been-deprecated)
    - [`Microsoft.AspNetCore.App` is now a FrameworkReference](#microsoftaspnetcoreapp-is-now-a-frameworkreference)
    - [The MvcPrecompilation tool has been deprecated](#the-mvcprecompilation-tool-has-been-deprecated)
    - [`Microsoft.AspNetCore.Server.Kestrel.Https` was removed](#microsoftaspnetcoreserverkestrelhttps-was-removed)
    - [Obselete Session APIs removed](#obselete-session-apis-removed)
    - [Authentication changes](#authentication-changes)
    - [`AllowSynchronousIO` is now set to false by default](#allowsynchronousio-is-now-set-to-false-by-default)
    - [Runtime complication of Razor Views/Pages is not available by default anymore](#runtime-complication-of-razor-viewspages-is-not-available-by-default-anymore)
    - [IHostingEnvironment's and IApplicationLifetime's marked obsolete and replaced](#ihostingenvironments-and-iapplicationlifetimes-marked-obsolete-and-replaced)
    - [`ResourceManagerWithCultureStringLocalizer` class and `WithCulture` interface member are now obselete](#resourcemanagerwithculturestringlocalizer-class-and-withculture-interface-member-are-now-obselete)
    - [Generic Host only supports specific constructor injections in Startup](#generic-host-only-supports-specific-constructor-injections-in-startup)
    - [Changes to ResponseCaching](#changes-to-responsecaching)
    - [`UseSignalR` and `UseConnections` are now obselete](#usesignalr-and-useconnections-are-now-obselete)
    - [`DefaultHttpContext` is not extensible anymore](#defaulthttpcontext-is-not-extensible-anymore)
    - [AspNetCoreModule V1 removed from Windows Hosting Bundle](#aspnetcoremodule-v1-removed-from-windows-hosting-bundle)
  - [Blazor](#blazor)

## ASP.NET Core

### ASP.NET Core 3.0 will only run on .NET Core

Quoting **natemcmaster**:
> As announced on the .NET Blog earlier this month, .NET Framework will get fewer of the newer platform and language features that come to .NET Core moving forward, due to the in-place update nature of .NET Framework and the desire to limit changes there that might break existing applications. To ensure ASP.NET Core can fully leverage the improvements coming to .NET Core moving forward, ASP.NET Core will only run on .NET Core starting from 3.0. Moving forward, you can simply think of ASP.NET Core as being part of .NET Core.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/324)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/3753)

### `Microsoft.AspNetCore.All` has been deprecated

A new meta-package was introduced: `Microsoft.AspNetCore.App`, which doesn't include Json.NET, EF Core and CodeAnalysis.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/314)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/3418)

### `Microsoft.AspNetCore.App` is now a FrameworkReference

This is such a neat feature, being able to easily reference the framework without worrying about the version and without searching in NuGet is such a good thing.

Quoting **natemcmaster**:
> Most NuGet packages provide both compilation and runtime assets. Microsoft.NETCore.App and Microsoft.AspNetCore.App effectively only provide the first - compilation references. Users must install other runtime assets to make .NET Core apps work but this is not obvious or intuitive, and not always possible: for example, Azure Web Apps, AWS, Google Cloud, etc. This violates a reasonable expectation of using a NuGet package, and has been a continual source of confusion for users.

Usage:

```csharp
<ItemGroup>
   <FrameworkReference Include="Microsoft.AspNetCore.App" />
</ItemGroup>
```

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/325)
- [Details](https://github.com/aspnet/AspNetCore/issues/3612)

### The MvcPrecompilation tool has been deprecated

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/315)
- [Discussion](https://github.com/aspnet/Mvc/issues/8313)
- [Razor SDK](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/sdk?view=aspnetcore-3.0)

### `Microsoft.AspNetCore.Server.Kestrel.Https` was removed

Quoting **Tratcher**:
> In ASP.NET Core 2.1 the contents of Microsoft.AspNetCore.Server.Kestrel.Https.dll were moved to Microsoft.AspNetCore.Server.Kestrel.Core.dll. This was done in a non-breaking way using TypeForwardedTo attributes. In the next major release (3.0) this empty assembly will be removed.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/329)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/4228)

### Obselete Session APIs removed

```csharp
public void ConfigureServices(ServiceCollection services)
{
    services.AddSession(options =>
    {
        // Removed obsolete APIs
        options.CookieName = "SessionCookie";
        options.CookieDomain = "contoso.com";
        options.CookiePath = "/";
        options.CookieHttpOnly = true;
        options.CookieSecure = CookieSecurePolicy.Always;
        // new API
        options.Cookie.Name = "SessionCookie";
        options.Cookie.Domain = "contoso.com";
        options.Cookie.Path = "/";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    });
}
```

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/331)

### Authentication changes

- The old 1.x stack, which was obselete, has been removed.
- `Newtonsoft.Json` dependecy in `Microsoft.AspNetCore.Authentication` was removed, it now uses `System.Text.Json`.

Links:

- Announcements: [1](https://github.com/aspnet/Announcements/issues/337) and [2](https://github.com/aspnet/Announcements/issues/341)
- Pull requests: [1](https://github.com/aspnet/AspNetCore/pull/6504), [2](https://github.com/aspnet/AspNetCore/pull/4485) and [3](https://github.com/aspnet/AspNetCore/pull/7105)
- Discussions: [1](https://github.com/aspnet/AspNetCore/issues/6533), [2](https://github.com/aspnet/Extensions/issues/1062) and [3](https://github.com/aspnet/AspNetCore/issues/7289)
- Migration guide: [Use HttpContext authentication extensions](https://docs.microsoft.com/en-us/aspnet/core/migration/1x-to-2x/identity-2x?view=aspnetcore-2.2#use-httpcontext-authentication-extensions) 

### `AllowSynchronousIO` is now set to false by default

`AllowSynchronousIO` is an option added to servers (Kestrel, IIS, ...) in order to allow or not synchronous IO API calls such as `HttpRequest.Body.Read(?)`.

Starting 3.0, this options is set to false by default and so all synchronous calls will throw an `InvalidOperationException` with the message: `Synchronous IO APIs are disabled, see AllowSynchronousIO.`

Links:

- [Announcement issue](https://github.com/aspnet/Announcements/issues/342)
- [Discussion issue](https://github.com/aspnet/AspNetCore/issues/7644)

### Runtime complication of Razor Views/Pages is not available by default anymore

Since the ASP.NET Core framework doesn't depend on Roslyn anymore, compiling views and pages at runtime is not possible anymore.

A new package has been added later, `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`, which is the same behaviour. To use it, call `AddMvcRazorRuntimeCompilation` on an `IMvcBuilder`.

Links:

- [Announcement issue](https://github.com/aspnet/Announcements/issues/343)
- [Discussion issue](https://github.com/aspnet/AspNetCore/issues/7647)

### IHostingEnvironment's and IApplicationLifetime's marked obsolete and replaced

Quoting **Tratcher**:
> When Microsoft.Extensions.Hosting was introduced in 2.1 some types like IHostingEnvironment and IApplicationLifetime were copied from Microsoft.AspNetCore.Hosting. Some 3.0 changes cause apps to include both the Microsoft.Extensions.Hosting and Microsoft.AspNetCore.Hosting namespaces. Any use of those duplicate types causes an "ambiguous reference" compiler error when both namespaces are referenced.

Obsolete types (warning):

- Microsoft.Extensions.Hosting.IHostingEnvironment
- Microsoft.AspNetCore.Hosting.IHostingEnvironment
- Microsoft.Extensions.Hosting.IApplicationLifetime
- Microsoft.AspNetCore.Hosting.IApplicationLifetime
- Microsoft.Extensions.Hosting.EnvironmentName
- Microsoft.AspNetCore.Hosting.EnvironmentName

New types:

- Microsoft.Extensions.Hosting.IHostEnvironment
- Microsoft.AspNetCore.Hosting.IWebHostEnvironment : IHostEnvironment
- Microsoft.Extensions.Hosting.IHostApplicationLifetime
- Microsoft.Extensions.Hosting.Environments

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/344)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/7749)
- [Pull Request](https://github.com/aspnet/AspNetCore/pull/7725) - [Pull Request](https://github.com/aspnet/AspNetCore/pull/7725)

### `ResourceManagerWithCultureStringLocalizer` class and `WithCulture` interface member are now obselete

Since these are becoming obselete, users are encouraged to use `CurrentCulture` and `CurrentUICulture`.

Links:

- [Accouncement](https://github.com/aspnet/Announcements/issues/346)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/7756)
- [Pull Request](https://github.com/aspnet/Extensions/pull/1133)

### Generic Host only supports specific constructor injections in Startup

In 3.0, ASP.NET Core uses the new generic host, which you can see in `Program.cs`. With the old web host, users were able to add constructor dependencies in the Startup class since it was using two containers. With this new implementation, only `IHostEnvironment`, `IWebHostEnvironment` and `IConfiguration` are injectable since it only uses a single container.

Note that dependencies can still be injected in the `Startup.Configure` method.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/353)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/9337)

### Changes to ResponseCaching

"pubinternal" types are now fully internal:

- Microsoft.AspNetCore.ResponseCaching.Internal.CachedResponse
- Microsoft.AspNetCore.ResponseCaching.Internal.CachedVaryByRules
- Microsoft.AspNetCore.ResponseCaching.Internal.IResponseCache
- Microsoft.AspNetCore.ResponseCaching.Internal.IResponseCacheEntry
- Microsoft.AspNetCore.ResponseCaching.Internal.IResponseCachingKeyProvider
- Microsoft.AspNetCore.ResponseCaching.Internal.IResponseCachingPolicyProvider
- Microsoft.AspNetCore.ResponseCaching.Internal.MemoryResponseCache
- Microsoft.AspNetCore.ResponseCaching.Internal.ResponseCachingContext
- Microsoft.AspNetCore.ResponseCaching.Internal.ResponseCachingKeyProvider
- Microsoft.AspNetCore.ResponseCaching.Internal.ResponseCachingPolicyProvider

Since these types are now internal, `AddResponseCaching` doesn't add a default implementation of `IResponseCachingPolicyProvider` nor `IResponseCachingKeyProvider`.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/354)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/9442)

### `UseSignalR` and `UseConnections` are now obselete

With the introduction of the new endpoints system, SignalR should now be added there and so these methods/classes are now obselete:

- `UseSignalR`
- `UseConnections`
- `ConnectionsRouteBuilder`
- `HasRouteBuilder`

Links :

- [Announcement](https://github.com/aspnet/Announcements/issues/362)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/10754)

### `DefaultHttpContext` is not extensible anymore

As part of the performance improvements in ASP.NET Core 3.0, see [this](https://github.com/aspnet/HttpAbstractions/pull/999) and [this](https://github.com/aspnet/AspNetCore/pull/6424), `DefaultHttpContext` is now sealed.

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/338)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/6534)
- [Pull Request](https://github.com/aspnet/AspNetCore/pull/6504)

### AspNetCoreModule V1 removed from Windows Hosting Bundle

Links:

- [Announcement](https://github.com/aspnet/Announcements/issues/339)
- [Discussion](https://github.com/aspnet/AspNetCore/issues/7095)

## Blazor

Todo
