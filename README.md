# Session aspnetcore hacks

For this workshop we've got a couple of hacks for AspNetCore. Each hack has a detailed walkthrough and some challenges. You can choose which one you want to do.
The start project is always the one provided in this repository which is a *default aspnetcore empty* app with `swashbuckle.aspnetcore` added.

### index
- [Healtchecks (200)](#Healthchecks%20\(200\))
- [JWS Signed Data (200)](#JWS%20Signed%20Data%20\(200\))
- [NodeServices (300)](#NodeServices%20\(300\))
- [Mediatr (300)](#Mediatr%20\(300\))

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

### Walktrough `[2 pt]`
To create, validate and read Json Web Tokens we can use `Microsoft.IdentityModel.JsonWebTokens.JsonWebTokenHandler` which is found in the nuget package `System.IdentityModel.Tokens.Jwt` so add this nuget package to the project.

Now create a controller (i.e. `ExternalDataController`) in the controllers folder.
```csharp
using Microsoft.AspNetCore.Mvc;
//---

[ApiController]
[Route("api/[controller]")]
public class ExternalDataController : Controller
{
}
```

#### Creating a JWT
Let's start with creating a JWT.
Create an Controller action called `GetSignedData` to the controller.
```csharp
[HttpGet]
public IActionResult GetSignedData()
{ return Ok(); }
```

For creating a JWT we need to specify the
- secret to sign the JWT
- algorithm to use
- payload

Obviously we want some data we want to share with the client which can be read, but can't be changed when sending back to the server. Create a small _DTO_ with some properties (i.e. `KvkDetails`).

We need this data to be converted to a Json string so we can sign it later on using the `JsonWebTokenHandler`.

```csharp
using Newtonsoft.Json;
//---

var dataJson = JsonConvert.SerializeObject(yourDtoInstance);
```

Now let's make sure we can sign the payload with a secret only we know. The simplest security key is a symmetric security key (vs an [asymmetric security key](https://hackernoon.com/symmetric-and-asymmetric-encryption-5122f9ec65b1)) so let's use that. We also need to create `SigningCredentials` which consists of the `SecurityKey` and an algorithm. We are going to use the `HmacSha256` algorithm which can be used for signing with a secret.

```csharp
using Microsoft.IdentityModel.Tokens;
//---

var securityKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(Key));
signingCredentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);
```

The payload of a JWT is based on claims and is a _Json object_. In the [specification](https://tools.ietf.org/html/rfc7519) there are a couple of [default claims/properties](https://tools.ietf.org/html/rfc7519#section-4.1) defined of this _Json object_. You don't have to use these (signing will just work fine), but they will provide you an extra layer of security. They are:

- `iss` Issuer; the instance which issued this token (in this case us)
- `sub` Subject; the id/uri of the subject which this token describes
- `aud` Audience; the subject for whom this token is (in this case also us)
- `exp` Expiration Time; the timestamp(epoch) when this token is expired
- `nbf` Not before Time; the timestamp(epoch) from when the token can be used
- `iat` Issued at; the timestamp(epoch) when the token is issued

The `JsonWebTokenHandler` has specific overloads which does the mapping to these claims. Because of the added security (i.e. we don't want a token to be used a years later) we are going to use these standard claims and add our own `data` claim.

Let's create the token and return it in the controller:
```csharp
using Microsoft.IdentityModel.JsonWebTokens;
//---

var jsonWebTokenHandler = new JsonWebTokenHandler();
var token = jsonWebTokenHandler.CreateToken(new SecurityTokenDescriptor
{
    Issuer = "HacksIss",
    Audience = "HacksAud",
    Claims = new Dictionary<string, object>
    {
        { "data", dataJson }
    },
    SigningCredentials = signingCredentials
});

return Ok(token);
```

Let's try this now from swagger and verify the generated JWT on https://jwt.io.

#### Reading and validating a JWT
On a later stage or different (micro)service we can get the data given to the (javascript)client and by validating the signature and the claims when available know that the data hasn't changed and came from a trusted source.

To do that, let's add a new action:
```csharp
[HttpPost]
public IActionResult SubmitSignedData(string token)
{ Ok() }
```

For validating a JWT we can use `ValidateToken` of `JsonWebTokenHandler` with `TokenValidationParameters` as parameter.

For this case we want to validate next to the signature, the Issuer and Audience.
```csharp
var tokenValidationParameters = new TokenValidationParameters
{
    IssuerSigningKey = securityKey, // same as when creating
    ValidAudience = "HacksAud",
    ValidIssuer = "HacksIss",
};

var validationResult = jsonWebTokenHandler.ValidateToken(token, tokenValidationParameters);
```

To check whether the token is valid use `validationResult.IsValid`.
```csharp
if (validationResult.IsValid)
{
    return Ok("valid");
} else {
    return BadRequest("Invalid token!")
}
```

Getting the content of the JWT is really easy (but doesn't do validation):
```csharp
var payload = jsonWebTokenHandler.ReadJsonWebToken(token);
var yourDtoInstance = payload.GetPayloadValue<YourDto>("data");
```

Now return `yourDtoInstance` instead of `"valid"` and see everything work. Try tampering your JWT, no more F12 developer tools hacking ^^.

### Challenge
(300) `[2 pt]` Use an asymmetric key. So create with a private key and validate with a public key.
(400) `[1 pt]` Try to create a JWT which can be used by `Microsoft.AspNetCore.Authentication.JwtBearer` authentication to authenticate. (Tip, this requires a `sub` claim and some configuration when registering like the `OpenIdConnectConfiguration`)

## NodeServices (300)
AspNetCore has the possibility to easily invoke Javascript via Node. Example usages are:
- Prerender a javascript component on the server
- Render markdown to html with a node package
- Validate objects using javascript to share validations between server and client

To use Node in AspNetCore we can use `NodeServices` which handles Node instances and serializes parameters and return objects to and from a javascript method. The file must be a `commonjs` module (for Node) and you must export a function which has as the first argument a callback for the result to be able to do asynchronous logic in the Javascript:
```js
module.exports = function (callback, /* your arguments, can be multiple */) {
    callback(/* error */ null, /* result */ "working");
};
```

### Walkthrough
For this walkthrough we are going to validate an object using a simple javascript file.

Add NodeServices to the `services` in the startup.cs:
```csharp
services.AddNodeServices();
```
Add a simple model which we want to validate and store, for example:
```csharp
public class JsData
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

And create a _Controller_ with a simple _POST_ Action with the data we want to validate.

```csharp
[ApiController]
[Route("jsdata")]
public class JsDataController : Controller
{
    [HttpPost]
    public async Task<IActionResult> PostJsData(JsData data)
    {
        return Ok();
    }
}
```

To invoke javascript we can use `INodeServices`, add a constructor with `INodeServices` as argument.

Now add the logic to invoke Javascript with Node:
```csharp
var result = await nodeServices.InvokeAsync<bool>("js/validate.js", data);
```
Some remarks on this line are:
- The `var result` is the result from the callback
- You need to specify the return type (`bool` in this case) so NodeServices can deserialize the result.
- It looks for the javascript file relative to the current project root by default
- You can add as many arguments as you like :)

Finaly, create the `validate.js` file in `/js` or another place (just remember that you need to update the Invoke :P).

If you added the example `JsData` the validate.js could look like this:
```js
return function (callback, model) {
    var valid = !!model.name && model.age >= 18;

    callback(/* error */ null, valid);
};
```

Now update the controller to return the result and try it out :).
```csharp
if(!result)
{
    ModelState.AddModelError("data", "not valid from JS");
    return BadRequest(ModelState);
}
else
{
    return Ok();
}
```
### Challenge
- (300) `[1 pt]` Also return a detailed error message from the javascript
- (400) `[3 pt]` Create some client javascipt which uses the same logic.

**TIP** for clientside js use `UMD` which has some checks to make it possible to consume the module with `commonjs`, `AMD` and `globals` in the browser.
```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define([], factory);
    } else if (typeof module === 'object' && module.exports) {
        // Node. Does not work with strict CommonJS, but
        // only CommonJS-like environments that support module.exports,
        // like Node.
        module.exports = factory();
    } else {
        // Browser globals (root is window)
        root.globalValidate = factory();
    }
}(typeof self !== 'undefined' ? self : this, function () {

    // ################ DO YOUR EXPORTS HERE ############
    // Just return a value to define the module export.
    // This example returns an object, but the module
    // can return a function as the exported value.
    return function (callback, model) {
        var valid = !!model.name && model.age >= 18;

        callback(/* error */ null, valid);
    };
}));
```

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

(300) `[1 pt]` Add a [custom pipeline behaviour](https://github.com/jbogard/MediatR/wiki/Behaviors) for common infrastructure like logging, validation, transaction management exception handling, etc..
