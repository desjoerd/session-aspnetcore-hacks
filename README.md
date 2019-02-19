# Session aspnetcore hacks

For this workshop we've got a couple of hacks for AspNetCore. Each hack has a detailed walkthrough and a challenge. You can choose which one you want to do.
The start project is always the one provided in this repository which is a *default aspnetcore empty* app with `swashbuckle.aspnetcore` added.

## Healthchecks (200)
ASP.NET Core has built in Health Check Middleware and other libraries for reporting the health of your app. These health reports are exposed as HTTP endpoint and they can be configured for different scenarios.

Examples:
- Basic health probe; check the status of a deployed app, if it returns HTTP 200 OK
- Database/EntityFramework probe; run a query to indicate if the database responds normally
- Readiness probe; check if an app is functioning but not yet ready to receive requests
- Liveness probe; check if an app is functioning and responding to requests
- Metric-based probe; check the app memory usage

### Walkthrough `[1 pt]` 
- Install NuGet Package `AspNetCore.HealthChecks.UI`

- Add a folder HealthChecks to your solution, and create a class named "AvailableHealthCheck". This class will be our first Health Check implementation.
    > ![AvailableHealthCheck](images/healthchecks_available.png)

- Have the class implement the IHealthCheck interface:
    ```csharp
    public class AvailableHealthCheck : IHealthCheck
    {
        public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
        {
            return Task.FromResult(HealthCheckResult.Healthy());
        }
    }
    ```

- In Startup.cs, add the following lines
    - To enable Health Checks in ASP.NET Core, add the following service to the ConfigureServices method:
    ```csharp
    services.AddHealthChecks()
        .AddCheck<AvailableHealthCheck>("available", tags: new[] { "ui" });
    ```
    - To enable the User Interface service, add the following:
    ```csharp
    services.AddHealthChecksUI();
    ```

- In Startup.cs, add the following lines to Configure method:
    ```csharp
    app.UseHealthChecks("/health/ui", new HealthCheckOptions()
    {
        Predicate = _ => _.Tags.Contains("ui"),
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
    });
    ```

- To enable a standard UI for Health Checks, add the following code to Startup.cs, Configure method:
    ```csharp
    app.UseHealthChecksUI();
    ```

- Now add a new section to the appsettings.json file with the following contents:
    ```json
    {
        "HealthChecks-UI": {
            "HealthChecks": [
                {
                    "Name": "UI",
                    "Uri": "http://localhost:5000/health/ui"
                }
            ],
            "EvaluationTimeOnSeconds": 5,
            "MinimumSecondsBetweenFailureNotifications": 60
        }
    }
    ```

- Open a shell in the Project folder and run the app with command `dotnet run`. Go to the health probe endpoint to review the result: http://localhost:5000/health/ui

- Now go to http://localhost:5000/healthchecks-ui to review the Health Checks UI

### Challenge `[3 pts total]` 
- (200) `[1 pt]` Add a Health Check that randomly changes status to the health checks page
- (300) `[1 pt]` Add your own creative real-world scenario health check, perhaps a database check? Maybe an API?
- (300) `[1 pt]` Add a web hook to post status updates. For example you can use a Telegram chat

## JWS Signed Data (200)

JWS stands for Json Web Signature and is distributed as a Json Web Token (JWT).
A json webtoken consists of three parts which are `base64` encoded seperated by a `.`.

- The first part is the header which contains the type of token and the signature/encryption algorithm.
- The second part is the actual `json` payload which can be encrypted.
- The last part is the signature.

https://jwt.io contains a debugger for json web tokens.

Example:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

For this workshop we are going to use JWS tokens which is a Json Web Token with a signature. Almost every JWT used for authentication is a JWS, but can also be used to share data between services. Because of the signature the JWS can be used to share data between two (micro)services via an untrusted source (i.e. a Javascript web client) and cannot be tampered.

### Walktrough


### Challenge

## NodeServices (300)

### Challenge

## Mediatr (300)

How to avoid Dependency Injection Constructor Madness? That is one of many architectural questions one can lose one's mind over. You might recognize constructors like this:
```csharp
public DashboardController(
    ICustomerRepository customerRepository,
    IOrderService orderService,
    ICustomerHistoryRepository historyRepository,
    IOrderRepository orderRepository,
    IProductRespoitory productRespoitory,
    IRelatedProductsRepository relatedProductsRepository,
    ISupportService supportService,
    ILog logger
)
```

Well fear no more, because the Mediator Pattern comes to the Rescue!

> A mediator is an object that makes decisions on how and when objects interact with each other. It encapsulates the 'how' and coordinates execution based on state, the way it’s invoked or the payload you provide to it.

MediatR is an open source implementation of the mediator pattern that doesn’t try to do too much and performs no magic. It allows you to compose messages, create and listen for events using synchronous or asynchronous patterns. It helps to reduce coupling and isolate the concerns of requesting the work to be done and creating the handler that dispatches the work.

MediatR has two kinds of messages it dispatches:
- Request/response messages, dispatched to a single handler
- Notification messages, dispatched to multiple handlers

For more cool information about MediatR, see [this blogpost](https://blogs.msdn.microsoft.com/cdndevs/2016/01/26/simplifying-development-and-separating-concerns-with-mediatr/) or go to the [official wiki](https://github.com/jbogard/MediatR/wiki).

### Walkthrough `[1 pt]`

- Add the `MediatR.Extensions.Microsoft.DependencyInjection` NuGet package


- In Startup.cs, add the following line to ConfigureServices:
    ```csharp
    services.AddMediatR();
    ```

- Create a command class:
    ```csharp
    public class CreateAwesomenessCommand : IRequest<bool>
    {
        public string AwesomeQuote { get; set; }
    }
    ```

- Create the handler for this command:
    ```csharp
    public class CreateAwesomenessHandler : IRequestHandler<CreateAwesomenessCommand, bool>
    {
        public Task<bool> Handle(CreateAwesomenessCommand request, CancellationToken cancellationToken)
        {
            return Task.FromResult(!string.IsNullOrEmpty(request.AwesomeQuote));
        }
    }
    ```

- In the Controllers folder, add a new API Controller class
    ```csharp
    [ApiController]
    [Route("commands")]
    public class CommandController : Controller
    {
        private readonly IMediator mediator;

        public  CommandController(IMediator mediator)
        {
            this.mediator = mediator ?? throw new ArgumentNullException(nameof(mediator));
        }
        
        [HttpPost]
        public async Task<IActionResult> PostAwesomeness(CreateAwesomenessCommand createAwesomenessCommand)
        {
            var commandResult = await mediator.Send(createAwesomenessCommand);

            if (commandResult) return Ok();
            
            return BadRequest();
        }
    }
    ```

- Open a shell in the Project folder and run the app with command `dotnet run`. Go to http://localhost:5000/swagger and experiment with your new command

### Challenge

(300) `[1 pt]` Add a custom pipeline handler
