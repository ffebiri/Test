# Microservices - A practical guide
<p>
In years past most organizations have built their applications using the monolithic architecture, popularly known as `Spaghetti architecture` Where all applications are packed in a single bucket. However, with the modern change in technology and the quest of big organization to scale up their applications to meet their client's expectations microservices is gradually becoming the new order of the day.       

To get an idea of What **Microservices** is, you have to understand how  monolithic application is decomposed into small tiny micro applications which are packaged and deployed independently. 

This Manual will decompose the idea of microservices and a practical way to implement and integrate all independent component of your application. It will clear your understanding of how developers use microservices to scale their applications according to their need.

Now, before I get deep into  microservices, let's look at the loose ends of the architecture that prevailed before microservices, i.e. the **monolithic architecture**. 
In layman's terms, you can say that the monolithic architecture is similar to a big container wherein all the software components of an application are assembled together and tightly packaged.
<p/>

![image](https://github.com/ffebiri/Test/blob/master/Mono.png)
___
#### Challenges with the Monolithic Architecture ####

> * Unscalable - Applications cannot be scaled easily since each time the application needs to be updated, the complete system has to be rebuilt.
> * Blocks Continuous Development - Many features of an application cannot be built and deployed at the same time.
> * Inflexible - Monolithic applications cannot be built using different technologies.
> * Unreliable - If even one feature of the system does not work, then the entire system does not work.
> * Slow Development - Development in monolithic applications takes a lot of time to be built since each and every feature has to be built one after the other.
> * Not Fit for Complex Applications - Features of complex applications have tightly coupled dependencies.
#### Microservices
<p>
How Different is microservices from Monolithic?
The Wikipedia article has a pretty good basic definition which begins with:

>  Microservices is a variant of the service-oriented architecture (SOA), an architectural style that structures an application as a collection of loosely coupled services. In a microservices architecture, services should be fine-grained and the protocols should be lightweight. 

In this writeup we will define microservice as *an architectural style that structures an application as a collection of small **autonomous** services, modeled around a business domain*.
<p/>

#### Why Choose Microservices?
> * Decoupling - Services within a system are largely decoupled, so the application as a whole can be easily built, altered, and scaled.
> * Componentization - Microservices are treated as autonomous components that can be easily replaced and upgraded.
> * Business Capabilities - Microservices are very simple and focus on a single capability.
> * Autonomy - Developers and teams can work independently of each other, thus increasing speed.
> * Continous Delivery - Allows frequent releases of software through systematic automation of software creation, testing, and approval.
> * Responsibility - Microservices do not focus on applications as projects. Instead, they treat applications as products for which they are responsible.
> * Decentralized Governance - The focus is on using the right tool for the right job. That means there is no standardized pattern or any technology pattern. Developers have the freedom to choose the best useful tools to solve their problems.
> * Agility - Microservices support agile development. Any new feature can be quickly developed and discarded again.
              
#### Typical Microservices 
---

![image](https://github.com/ffebiri/Test/blob/master/Micros.png )
___

### Structure of Microservice
<p> We will at this point look at a single microservice as a collection of related functions or endpoints composing a bounded context within your business domain. Each microservice should be bound in such a way that it can own its own data and operate autonomously, communicating with other microservices via lightweight protocols only.
<p/>

___

>* Microservice as an autonomouse unit

![image](https://github.com/ffebiri/Test/blob/master/Microservice.PNG)
(Keeping Things Really Simple) 
* API level can be inferred in the program as MVC going to controller. Again it is the only part of the microservice visible on the outside.
* BL - The work that needs to be programmed (connection to DB)
* Models: Used to manipulate database (working with DB)
* Config – Configuration of the whole services

___

>  **Points to note at this level**
> > * As mentioned above, the API is the only visible part from outside (You can only call the API)
> > * What we want at this point is  to containerised this service, to do that in Docker, we will rap bubbles around the entire service and expose the API through the bubble.
> > * At this point it becomes clear that our config will be hidden in the rapped bubbles and do not neccessarily forms part of our service.
> > * configs are going to be supply from the bubbles we have rapped the service in and not from the service itself. That is to say, configs are going to be imported when starting or running the container.(coming from docker)
> > * A practical reason behind this scenario above is that,if you move your DB to somewhere else, you don't need to go back to your code and change anything or build and test your application again, all you need is to replace the old config with new config. 
 [Read more on Docker container Config](https://docs.docker.com/edge/engine/reference/commandline/config/).


### Inter service Communication
<p> Microservices-based application  is a distributed system running on multiple processes or services, usually even across multiple servers or hosts. Each service instance is typically a process. Therefore, services must interact using an inter-process communication protocol.I will briefly discuss using Asynchonous Communication through a Message Queue<p/>

### Asynchronous Communication
<p>Why Asynchronous ? - The key point we want to establish  here is that the client should not have blocked a thread while waiting for a response. It doesn't really have to wait for a response. See this way of communication as a one way communication. Example, from the diagrm below service A sends a request to  MQ  and doesn't have to wait for response, service B sends request to MQ and doesn't also have to wait for response, This one way of communication exist between all services. It just waits for acknowledgment that the message has been received by the Message Queue. <p/>

---

![image](https://github.com/ffebiri/Test/blob/master/async.PNG)
---
### KEY ISSUES 
>* The question that is likely unanswered at this point is What happens if one service needs some real time data from another service?
>>* A straight answer to this, In microservices,Your individual services must be structured in such a way that they never need data from another service.Each service is an independent from others.

On the contrary, combination of data from multiple services is possible from the frontend. The monolithic architecture will Call service A, service A fetches data from service B and display the results. With microservices, the trend is quite different, place a call to service A and  Call service B seperately, then combine the result from service A and results from service B on the front-end.
>* Another Key issue is How do we call third party service eg. SAP
>>*  Third part services mostly will be called directly or you can also use the integration service through the message queue.


### Practical Microservice Template
<p>Now let us take a detail look into how a single service looks like in microservice. We will be looking at a detail template i have preapared specially for this project.This will help you to understand how you can configure your service. We will be taking Detail analysis of our Program.cs and startup.cs classes for now.
<p/>

>  #### Program.cs

```
using System;
using System.IO;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Serilog;
using Serilog.Events;
using Serilog.Sinks.Elasticsearch;

namespace Orion.Core.Template
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                //IHostingEnvironment env = hostingContext.HostingEnvironment;
                 var pathToContentRoot = Directory.GetCurrentDirectory();

                config.SetBasePath(pathToContentRoot)
                .AddEnvironmentVariables()
                // basic settings
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                // log db connection string + settings?
                .AddJsonFile("logsettings.json", optional: true, reloadOnChange: true)
                // database connection strings
                .AddJsonFile("databasesettings.json", optional: true, reloadOnChange: true);
            })
            .UseStartup<Startup>()
            .UseSerilog((context, loggerConfig) =>
            {
                var connectionString = context.Configuration.GetConnectionString("LogConnection");

                loggerConfig.MinimumLevel.Information()
                    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
                    .Enrich.FromLogContext()
                    .Enrich.WithProperty("Application", typeof(Program))
                    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri(connectionString))
                 {
                     AutoRegisterTemplate = true
                 });
            });
    }
}

```

### The Main Method
```
public static void Main(string[] args)
{
  CreateWebHostBuilder(args).Build().Run();
}
```
The Main method is the entry point to a C#  application . When the application is started, the Main method is the first method that is invoked. Say this is where all the magic start.[Learn more about the main method](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/hello-world-your-first-program "Main() ")

### Set up a Host- Configuration

In the `Main` method I have [CreateWebHostBuilder method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createwebhostbuilder?view=aspnetcore-2.1).This method create a host using an instance of [IWebHostBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder?view=aspnetcore-2.1) which creates configuration for the server.
This is typically performed in the app's entry point, the `Main` method. In the project templates, `Main` is located in Program.cs. A typical Program.cs calls `CreateDefaultBuilder()` to start setting up a host: [CreateDefaultBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder?view=aspnetcore-2.1) .`CreateDefaultBuilder()` Initializes a new instance of the
 [WebHostBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder?vie"WebHostBuilder) class with pre-configured defaults.

[ConfigureAppConfiguration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureappconfiguration?view=aspnetcore-2.1) Adds a delegate for configuring the [IConfigurationBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder?view=aspnetcore-2.1) that will construct an [IConfiguration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration?view=aspnetcore-2.1). In future adding secrets and other sensitive data can be done from this point. Again Environment variables can also be set at this point.
```
        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
                WebHost.CreateDefaultBuilder(args)
                    .ConfigureAppConfiguration((hostingContext, config) =>
                    {
                        //IHostingEnvironment env = hostingContext.HostingEnvironment;
                        var pathToContentRoot = Directory.GetCurrentDirectory();

                        config.SetBasePath(pathToContentRoot)
                            .AddEnvironmentVariables()
                            // basic settings
                            .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                            // log db connection string + settings?
                            .AddJsonFile("logsettings.json", optional: true, reloadOnChange: true)
                            // database connection strings
                            .AddJsonFile("databasesettings.json", optional: true, reloadOnChange: true);
                    })
                    .UseStartup<Startup>()
```

Take note of the `ConfigureAppConfiguration` function call:
The function takes a single argument, which is a callback that takes a `WebHostBuilderContext` and an `IConfigurationBuilder`.
You can configure configuration as usual in there. 
> Tip: Note that calling `ConfigureAppConfiguration` again after creating the default builder does not replace the default configuration, but add on top of it.

**But wait!** How can we access the built configuration in Startup?

The new Startup constructor looks like this:

```
public Startup(IConfiguration configuration)
{
     Configuration = configuration;
}
public IConfiguration Configuration { get; }

```
>*  This section will be dicussed in more detailed in the next documentation
### Configure Services

```
       public void ConfigureServices(IServiceCollection services)
            {
                // DB context prepared for multitenancy
                services.AddDbContext<ApplicationDbContext>(options => { });

                // This client is needed only if you are going to send some data
                services.AddHttpClient();

                // load db settings + join them with secrets
                // Todo: Join connection strings with secrets later
                services.Configure<TenantMiddlewareConfig>(settings =>
                {
                    settings.ConnectionStrings = Configuration.Get<TenantMiddlewareConfig>().ConnectionStrings;
                });

                // Scoped "variable" used for storing correct connection string
                services.AddScoped<ITenantProvider, DefaultTenantProvider>();

                // todo: how is it going to be in dev/tst/prod - change once I have IDP installed(Currently using Json web token)
                services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
                .AddJwtBearer(options =>
                {
                    options.TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateIssuer = true,
                        ValidateAudience = true,
                        ValidateLifetime = true,
                        ValidateIssuerSigningKey = true,
                        // here should be correct address/name of identity provider
                        ValidIssuer = "identityProvider.orion.com",
                        // here goes name of this current API
                        ValidAudience = "JWTMiddleware.orion.com",
                        // this is going to change once we have IDP server ready
                        IssuerSigningKey = new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes("TopSecretKeyForThisApplication"))
                    };
                });

                services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
            }

            public void Configure(IApplicationBuilder app, IHostingEnvironment env)
            {
                if (env.IsDevelopment())
                {
                    app.UseDeveloperExceptionPage();
                }

                // this order is necessary, do not change it!
                app.UseAuthentication();
                app.UseTenantMiddleware();
                app.UseMvc();
            }  

```
The [ConfigureServices](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configureservices?view=aspnetcore-2.1) method is:
 * Optional
 * Called by the web host before the `Configure` method to configure the app's services.
 * Where [configuration options](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/index?view=aspnetcore-2.1) are set by convention.

The typical pattern is to call all the `Add{Service}` methods, and then call all the `services.Configure{Service}` methods.

Adding services to the service container makes them available within the app and in the Configure method. The services are resolved via `dependency injection` or from `IApplicationBuilder.ApplicationServices`.



I have optionally included the `ConfigureServices` method to configure the app's services. The methods must come inline with the [Configure](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configure?view=aspnetcore-2.1) method to create the app's request pipeline. `ConfigureServices` and `Configure` are called by the runtime when the app starts:

In the template the following backgroud services have been added (**Details of these services will be treated in later version on this manual**)

>* DbContext (Registers the given DB context as a service in the `IServiceCollection`.)
```
    services.AddDbContext<ApplicationDbContext>(options => { });
```
>* HttpClient
```
    services.AddHttpClient();
```
>* Tenant Configurations(`load db settings + join them with secrets`. 
 `Join connection strings with secrets later`)

```
     services.Configure<TenantMiddlewareConfig>(settings =>
     {
        settings.ConnectionStrings = Configuration.Get<TenantMiddlewareConfig>().ConnectionStrings;
     });
```


>* AddScoped (Creates a new instance for each http request.)

```
     services.AddScoped<ITenantProvider, DefaultTenantProvider>();
```
>* Authentication ()

```
       services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
       .AddJwtBearer(options =>
        {
          options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            // here should be correct address/name of identity provider
            ValidIssuer = "identityProvider.orion.com",
            // here goes name of this current API
            ValidAudience = "JWTMiddleware.orion.com",
            // this is going to change once we have IDP server ready
            IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes("TopSecretKeyForThisApplication"))
        };
        });

```
>* AddMvc

```
   services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
   (we set our compactibilityversion to 2.1 to allow the service opt_in of potentially behaviour changes introduced by ASP.NET CORE MVC 2.1 or later)
```
### The Configure method

```
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
            {
                if (env.IsDevelopment())
                {
                    app.UseDeveloperExceptionPage();
                }

                // this order is necessary, do not change it!
                app.UseAuthentication();
                app.UseTenantMiddleware();
                app.UseMvc();
            }
```
ASP.NET Core reads the environment variable `ASPNETCORE_ENVIRONMENT` at app `startup` and stores the value in `IHostingEnvironment.EnvironmentName`. 

```
if (env.IsDevelopment())
 {
    app.UseDeveloperExceptionPage();
 }

```
<p>The preceding code:

Calls `UseDeveloperExceptionPage` and `UseBrowserLink` when `ASPNETCORE_ENVIRONMENT` is set to `Development`.<p/>


The `Configure` method is used to specify the order the app responds to HTTP requests. The request pipeline is configured by adding middleware components to an `IApplicationBuilder instance`. 

 Each `Use` extension method adds a middleware component to the request pipeline. 
 From our template  it is important to note that each of our middleware component is responsible for invoking the next component in the pipeline. It is therefore important to note that the order. 

Our startup class will therefore look like this..

>**Startup.cs**

```
    using System.Text;
    using Microsoft.AspNetCore.Authentication.JwtBearer;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.IdentityModel.Tokens;
    using Orion.Core.MultiTenantMiddleware;
    using Orion.Core.Template.Data;

    namespace Orion.Core.Template
    {
        public class Startup
        {
            public IConfiguration Configuration { get; }

            public Startup(IConfiguration configuration)
            {
                Configuration = configuration;
            }

            public void ConfigureServices(IServiceCollection services)
            {
               
                services.AddDbContext<ApplicationDbContext>(options => { });

                services.AddHttpClient();

                services.Configure<TenantMiddlewareConfig>(settings =>
                {
                    settings.ConnectionStrings = Configuration.Get<TenantMiddlewareConfig>().ConnectionStrings;
                });

                services.AddScoped<ITenantProvider, DefaultTenantProvider>();

                services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
                .AddJwtBearer(options =>
                {
                    options.TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateIssuer = true,
                        ValidateAudience = true,
                        ValidateLifetime = true,
                        ValidateIssuerSigningKey = true,
                        ValidIssuer = "identityProvider.orion.com",
                        ValidAudience = "JWTMiddleware.orion.com",
                        IssuerSigningKey = new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes("TopSecretKeyForThisApplication"))
                    };
                });

                services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
            }

            public void Configure(IApplicationBuilder app, IHostingEnvironment env)
            {
                if (env.IsDevelopment())
                {
                    app.UseDeveloperExceptionPage();
                }
                app.UseAuthentication();
                app.UseTenantMiddleware();
                app.UseMvc();
            }
        }
    }
```







