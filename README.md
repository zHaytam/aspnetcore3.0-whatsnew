# ASP.NET Core 3.0 and Blazor

This file/repository is aimed to group all the breaking changes and new features that got introduced in version 3.0 for a quick read/search.

## Table of content

- [ASP.NET Core 3.0 and Blazor](#aspnet-core-30-and-blazor)
  - [Table of conten$$t](#table-of-content)
  - [SignalR](#signalr)
    - [Client-to-server streaming](#client-to-server-streaming)
      - [Example (source)](#example-source)
    - [Long polling for Java SignalR Client](#long-polling-for-java-signalr-client)
    - [New Customizable SignalR Hub Method Authorization](#new-customizable-signalr-hub-method-authorization)
      - [Example source](#example-source)
  - [ASP.NET Core](#aspnet-core)
    - [System.IO.Pipelines on HttpContext](#systemiopipelines-on-httpcontext)
      - [Example (source)](#example-source-1)
    - [Generic host templates](#generic-host-templates)
    - [Endpoint Routing](#endpoint-routing)
    - [Worker Service](#worker-service)
    - [gRPC](#grpc)
    - [AddMvc() was split](#addmvc-was-split)
      - [AddControllers](#addcontrollers)
      - [AddControllersWithViews](#addcontrollerswithviews)
      - [AddRazorPages](#addrazorpages)
    - [New JSON Serialization](#new-json-serialization)
    - [Certificate and Kerberos authentication](#certificate-and-kerberos-authentication)
      - [Example source](#example-source-1)
    - [EventCounters](#eventcounters)
      - [Hosting (`Microsoft.AspNetCore.Hosting`)](#hosting-microsoftaspnetcorehosting)
      - [SignalR (`Microsoft.AspNetCore.Http.Connections`)](#signalr-microsoftaspnetcorehttpconnections)
      - [gRPC (`Grpc.AspNetCore.Server` and `Grpc.Net.Client`)](#grpc-grpcaspnetcoreserver-and-grpcnetclient)
      - [Viewing](#viewing)
    - [Razor Pages support for `@attribute`](#razor-pages-support-for-attribute)
    - [New networking primitives for non-HTTP Servers](#new-networking-primitives-for-non-http-servers)
      - [Example source](#example-source-2)
    - [Unix domain socket support for the Kestrel Sockets transport](#unix-domain-socket-support-for-the-kestrel-sockets-transport)
  - [Blazor](#blazor)
    - [Culture aware data binding](#culture-aware-data-binding)
    - [Case-sensitive component binding](#case-sensitive-component-binding)
    - [Accepting arbitrary parameters](#accepting-arbitrary-parameters)
    - [Authentication & Authorization](#authentication--authorization)
    - [Static assets](#static-assets)
    - [Forms & Validation](#forms--validation)
      - [Example source](#example-source-3)
    - [Razor Components in Razor Class Libraries](#razor-components-in-razor-class-libraries)
    - [Prerendering](#prerendering)
    - [Reconnecting in Blazor (server-side)](#reconnecting-in-blazor-server-side)
    - [Render stateful interactive components from Razor pages and views](#render-stateful-interactive-components-from-razor-pages-and-view
  - [Breaking Changes](#breaking-changes)
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

## SignalR

### Client-to-server streaming

In Preview2, a new feature has been added to SignalR which gives the ability for the server to listen to streams.

#### Example ([source](https://devblogs.microsoft.com/aspnet/aspnet-core-3-preview-2/))

Server (Hub):

```csharp
public async Task StartStream(string streamName, ChannelReader<string> streamContent)
{
    // read from and process stream items
    while (await streamContent.WaitToReadAsync(Context.ConnectionAborted))
    {
        while (streamContent.TryRead(out var content))
        {
            // process content
        }
    }
}
```

Client:

```js
let subject = new signalR.Subject();
await connection.send("StartStream", "MyAsciiArtStream", subject);
subject.next("example");
subject.complete();
```

For more information, please check the [documentation page](https://docs.microsoft.com/en-us/aspnet/core/signalr/streaming?view=aspnetcore-2.2).

### Long polling for Java SignalR Client

> We added long polling support to the Java client which enables it to establish connections even in environments that do not support WebSockets. This also gives you the ability to specifically select the long polling transport in your client apps.

### New Customizable SignalR Hub Method Authorization

> SignalR now provides a custom resource to authorization handlers when a hub method requires authorization. The resource is an instance of HubInvocationContext. The HubInvocationContext includes the HubCallerContext, the name of the hub method being invoked, and the arguments to the hub method.

#### Example [source](https://devblogs.microsoft.com/aspnet/asp-net-core-and-blazor-updates-in-net-core-3-0-preview-7/)

```csharp
public class DomainRestrictedRequirement :
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>,
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement,
        HubInvocationContext resource)
    {
        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name) &&
            context.User != null &&
            context.User.Identity != null &&
            context.User.Identity.Name.EndsWith("@jabbr.net", StringComparison.OrdinalIgnoreCase))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName,
        string currentUsername)
    {
        return !(currentUsername.Equals("bob42@jabbr.net", StringComparison.OrdinalIgnoreCase) &&
            hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase));
    }
}

[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}

// Startup.cs
services
    .AddAuthorization(options =>
    {
        options.AddPolicy("DomainRestricted", policy =>
        {
            policy.Requirements.Add(new DomainRestrictedRequirement());
        });
    });
```

## ASP.NET Core

### System.IO.Pipelines on HttpContext

The body request and response pipes on the `HttpContext` are now exposed to users looking to write high performance code.

These pipes are using the new [System.IO.Pipelines API](https://blogs.msdn.microsoft.com/dotnet/2018/07/09/system-io-pipelines-high-performance-io-in-net/).

#### Example ([source](https://devblogs.microsoft.com/aspnet/aspnet-core-3-preview-2/))

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting(routes =>
    {
        routes.MapGet("/", async context =>
        {
            await context.Response.WriteAsync("Hello World");
        });

        routes.MapPost("/", async context =>
        {
            while (true)
            {
                var result = await context.Request.BodyPipe.ReadAsync();
                var buffer = result.Buffer;

                if (result.IsCompleted)
                {
                    break;
                }

                context.Request.BodyPipe.AdvanceTo(buffer.End);
            }
        });
    });
}
```

### Generic host templates

ASP.NET Core now uses the [Generic Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.2) by default.

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Note that this comes with a [breaking change](#generic-host-only-supports-specific-constructor-injections-in-startup).

### Endpoint Routing

Quoting **Daniel** from this blog [post](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-core-3-0-preview-4/):
> Endpoint Routing is made up of the pair of middleware created by app.UseRouting() and app.UseEndpoints(). app.UseRouting() marks the position in the middleware pipeline where a routing decision is made – where an endpoint is selected. app.UseEndpoints() marks the position in the middleware pipeline where the selected endpoint is executed. Middleware that run in between these can see the selected endpoint (if any) or can select a different endpoint.

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Middleware that run before routing. Usually the following appear here:
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");

    }

    app.UseStaticFiles();

    // Runs matching. An endpoint is selected and set on the HttpContext if a match is found.
    app.UseRouting();

    // Middleware that run after routing occurs. Usually the following appear here:
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseCors();
    // These middleware can take different actions based on the endpoint.

    // Executes the endpoint that was selected by routing.
    app.UseEndpoints(endpoints =>
    {
        // Mapping of endpoints goes here:
        endpoints.MapControllers();
        endpoints.MapRazorPages();
        endpoints.MapHub<MyChatHub>();
        endpoints.MapGrpcService<MyCalculatorService>();
    });

    // Middleware here will only run if nothing was matched.
}
```

### Worker Service

ASP.NET Core adds a new template for worker services for long running tasks such as Windows Services or Linux Systemd daemons.

### gRPC

A new project template has been added for users interested in creating [gRPC](https://grpc.io/) servers, which use HTTP/2 for transport and protocol buffers, and supports HTTPs by default.

A package `Grpc.Net.ClientFactory` is available for developers interested in a gRPC Client Factory. It's intented to use in scenarios that don't include ASP.NET Core.

The package also gives the possibility to include interceptors.

```csharp
services
    .AddGrpcClient<GreeterClient>(options =>
    {
        options.BaseAddress = new Uri("https://localhost:5001");
    })
    .AddInterceptor<CallbackInterceptor>();
```

gRPC also supports `CallCredentials`, allowing for interoperability with existing libraries like `Grpc.Auth` that rely on `CallCredentials`.

Links:

- [gRPC dotnet repository](https://github.com/grpc/grpc-dotnet)

### AddMvc() was split

In order to give developpers more option in what they include in their projects, `AddMvc` was split into 3 top level extension methods:

#### AddControllers

- Controllers
- Model Binding
- API Explorer (OpenAPI integration)
- Authorization [Authorize]
- CORS [EnableCors]
- Data Annotations validation [Required]
- Formatter Mappings (translate a file-extension to a content-type)

#### AddControllersWithViews

- Controllers
- Model Binding
- API Explorer (OpenAPI integration)
- Authorization [Authorize]
- CORS [EnableCors]
- Data Annotations validation [Required]
- Formatter Mappings (translate a file-extension to a content-type)
- Antiforgery
- Temp Data
- Views
- Tag Helpers
- Memory Cache

#### AddRazorPages

- Pages
- Controllers
- Model Binding
- Authorization [Authorize]
- Data Annotations validation [Required]
- Antiforgery
- Temp Data
- Views
- Tag Helpers
- Memory Cache

### New JSON Serialization

The .NET Core team introduced a new JSON Serialization library, found in the `System.Text.Json` namespace.

> For the most common payload sizes, System.Text.Json offers about 20% throughput increase during input and output formatting with a smaller memory footprint.

It is now used by default in ASP.NET Core projects and SignalR Hubs.

### Certificate and Kerberos authentication

> Preview 6 brings Certificate and Kerberos authentication to ASP.NET Core.  
Certificate authentication requires you to configure your server to accept certificates, and then add the authentication middleware in Startup.Configure and the certificate authentication service in Startup.ConfigureServices.

#### Example [source](https://devblogs.microsoft.com/aspnet/asp-net-core-and-blazor-updates-in-net-core-3-0-preview-6/)

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate();
    // All the other service configuration.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // All the other app configuration.
}
```

### EventCounters

> In place of Windows perf counters, .NET Core introduced a new way of emitting metrics via [EventCounters](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventcounter?view=netcore-3.0).

#### Hosting (`Microsoft.AspNetCore.Hosting`)

- requests-per-second
- total-requests
- current-requests
- failed-requests

#### SignalR (`Microsoft.AspNetCore.Http.Connections`)

- connections-started
- connections-stopped
- connections-timed-out
- connections-duration

#### gRPC (`Grpc.AspNetCore.Server` and `Grpc.Net.Client`)

- total-calls
- current-calls
- calls-failed
- calls-deadline-exceeded
- messages-sent
- messages-received
- calls-unimplemented

#### Viewing

`dotnet counters monitor -p <PID> Microsoft.AspNetCore.Hosting System.Runtime`

### Razor Pages support for `@attribute`

This is a feature that was added to Blazor that lets developers include attributes (e.g. Authorize) on their components. It is now also available in Razor Pages!

### New networking primitives for non-HTTP Servers

> As part of the effort to decouple the components of Kestrel, we are introducing new networking primitives allowing you to add support for non-HTTP protocols.

> You can bind to an endpoint (System.Net.EndPoint) by calling Bind on an IConnectionListenerFactory. This returns a IConnectionListener which can be used to accept new connections. Calling AcceptAsync returns a ConnectionContext with details on the connection. A ConnectionContext is similar to HttpContext except it represents a connection instead of an HTTP request and response.

#### Example [source](https://devblogs.microsoft.com/aspnet/asp-net-core-and-blazor-updates-in-net-core-3-0-preview-8/)

```csharp
public class TcpEchoServer : BackgroundService
{
    private readonly ILogger<TcpEchoServer> _logger;
    private readonly IConnectionListenerFactory _factory;
    private IConnectionListener _listener;

    public TcpEchoServer(ILogger<TcpEchoServer> logger, IConnectionListenerFactory factory)
    {
        _logger = logger;
        _factory = factory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _listener = await _factory.BindAsync(new IPEndPoint(IPAddress.Loopback, 6000), stoppingToken);

        while (true)
        {
            var connection = await _listener.AcceptAsync(stoppingToken);
            // AcceptAsync will return null upon disposing the listener
            if (connection == null)
            {
                break;
            }
            // In an actual server, ensure all accepted connections are disposed prior to completing
            _ = Echo(connection, stoppingToken);
        }

    }

    public override async Task StopAsync(CancellationToken cancellationToken)
    {
        await _listener.DisposeAsync();
    }

    private async Task Echo(ConnectionContext connection, CancellationToken stoppingToken)
    {
        try
        {
            var input = connection.Transport.Input;
            var output = connection.Transport.Output;

            await input.CopyToAsync(output, stoppingToken);
        }
        catch (OperationCanceledException)
        {
            _logger.LogInformation("Connection {ConnectionId} cancelled due to server shutdown", connection.ConnectionId);
        }
        catch (Exception e)
        {
            _logger.LogError(e, "Connection {ConnectionId} threw an exception", connection.ConnectionId);
        }
        finally
        {
            await connection.DisposeAsync();
            _logger.LogInformation("Connection {ConnectionId} disconnected", connection.ConnectionId);
        }
    }
```

### Unix domain socket support for the Kestrel Sockets transport

Kestrel now supports Unix domain sockets (on Linux, macOS, and Windows 10, version 1803 and newer).

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder
                .ConfigureKestrel(o =>
                {
                    o.ListenUnixSocket("/var/listen.sock");
                })
                .UseStartup<Startup>();
        });
```

## Blazor

### Culture aware data binding

> Data-binding support (@bind) for `<input>` elements is now culture-aware. Data bound values will be formatted for display and parsed using the current culture as specified by the System.Globalization.CultureInfo.CurrentCulture property. This means that @bind will work correctly when the user’s desired culture has been set as the current culture, which is typically done using the ASP.NET Core localization middleware (see Localization).

> You can also manually specify the culture to use for data binding using the new @bind:culture parameter, where the value of the parameter is a CultureInfo instance. For example, to bind using the invariant culture:

`<input @bind="amount" @bind:culture="CultureInfo.InvariantCulture" />
`

### Case-sensitive component binding

> Components in .razor files are now case-sensitive. This enables some useful new scenarios and improves diagnostics from the Razor compiler.

> For example, the Counter has a button for incrementing the count that is styled as a primary button. What if we wanted a Button component that is styled as a primary button by default? Creating a component named Button in previous Blazor releases was problematic because it clashed with the button HTML element, but now that component matching is case-sensitive we can create our Button component and use it in Counter without issue.

### Accepting arbitrary parameters

> Components can now capture and render additional attributes in addition to the component’s declared parameters. Additional attributes can be captured in a dictionary and then “splatted” onto an element as part of the component’s rendering using the new @attributes Razor directive. This feature is especially valuable when defining a component that produces a markup element that supports a variety of customizations. For instance if you were defining a component that produces an `<input>` element, it would be tedious to define all of the attributes `<input>` supports like maxlength or placeholder as component parameters.

```csharp
<input class="form-field" @attributes="Attributes" type="text" />

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    Dictionary<string, object> Attributes { get; set; }
}
```

In the above example:

- If `Attributes` contains a class then it will be used instead of `form-field`.
- If `Attributes` contains a type then it will be overwritten by `text` since it comes after.
- All other attributes are simply added.

### Authentication & Authorization

> Blazor now has built-in support for handling authentication and authorization. The server-side Blazor template now supports options for enabling all of the standard authentication configurations using ASP.NET Core Identity, Azure AD, and Azure AD B2C. We haven’t updated the Blazor WebAssembly templates to support these options yet, but we plan to do so after .NET Core 3.0 has shipped.

```html
<AuthorizeView>
    <Authorized>
        <a href="Identity/Account/Manage">Hello, @context.User.Identity.Name!</a>
        <a href="Identity/Account/LogOut">Log out</a>
    </Authorized>
    <NotAuthorized>
        <a href="Identity/Account/Register">Register</a>
        <a href="Identity/Account/Login">Log in</a>
    </NotAuthorized>
</AuthorizeView>
```

### Static assets

> Razor class libraries can now include static assets like JavaScript, CSS, and images. These static assets can then be included in ASP.NET Core apps by referencing the Razor class library project or via a package reference.

Static assets must be in the `wwwroot` folder.

### Forms & Validation

Blazor now supports forms & validation!

#### Example [source](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-core-3-0-preview-3/)

Model:

```csharp
public class Person
{
    [Required(ErrorMessage = "Enter a name")]
    [StringLength(10, ErrorMessage = "That name is too long")]
    public string Name { get; set; }

    [Range(0, 200, ErrorMessage = "Nobody is that old")]
    public int AgeInYears { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", ErrorMessage = "Must accept terms")]
    public bool AcceptsTerms { get; set; }
}
```

UI:

```csharp
<EditForm Model="@person" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p class="name">
        Name: <InputText bind-Value="@person.Name" />
    </p>
    <p class="age">
        Age (years): <InputNumber bind-Value="@person.AgeInYears" />
    </p>
    <p class="accepts-terms">
        Accepts terms: <InputCheckbox bind-Value="@person.AcceptsTerms" />
    </p>

    <button type="submit">Submit</button>
</EditForm>

@functions {
    Person person = new Person();

    void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

### Razor Components in Razor Class Libraries

You can now create Razor Components in Razor Class Libraries and use them from ASP.NET Core projects.

### Prerendering

> The Razor Components project template now does server-side prerendering by default. This means that when a user navigates to your application, the server will perform an initial render of your Razor Components and deliver the result to their browser as plain static HTML. Then, the browser will reconnect to the server via SignalR and switch the Razor Components into a fully interactive mode. This two-phase delivery is beneficial because:

- It improves the perceived performance of the site, because the UI can appear sooner, without waiting to make any WebSocket connections or even running any client-side script. This makes a bigger difference for users on slow connections, such as 2G/3G mobiles.
- It can make your application easily crawlable by search engines.

### Reconnecting in Blazor (server-side)

Since Blazor (server-side) requires SignalR to run, it will now attempt to reconnect whenever it gets disconnected.

The reconnection timings can be changed, for example:

```js
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .withAutomaticReconnect([0, 0, 2000, 5000]) // defaults to [0, 2000, 10000, 30000]
    .build();
```

Upong reconnecting, the `onreconnecting` callback is fired. When the reconnection is successfull, the `onreconnected` callback is fired.

### Render stateful interactive components from Razor pages and views

> You can now add stateful interactive components to a Razor page or View. When the page or view renders the component will be prerendered with it. The app will then reconnect to the component state once the client connection has been established as long as it is still in memory.

```csharp
<h1>My Razor Page</h1>
<form>
    <input type="number" asp-for="InitialCount" />
    <button type="submit">Set initial count</button>
</form>

@(await Html.RenderComponentAsync<Counter>(new { InitialCount = InitialCount }))

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialCount { get; set; }
}
```

![Result GIF](https://devblogs.microsoft.com/aspnet/wp-content/uploads/sites/16/2019/04/interactive-component-on-razor-page.gif)

## Breaking Changes

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
