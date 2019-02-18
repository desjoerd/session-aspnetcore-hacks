# Session aspnetcore hacks

For this workshop we've got a couple of hacks for AspNetCore. Each hack has a detailed walkthrough and a challenge. You can choose which one you want to do.
The start project is always the one provided in this repository which is a *default aspnetcore empty* app with `swashbuckle.aspnetcore` added.

## Healthchecks (200)
HealthChecks in ASP.NET Core are ......... [link]

### Walkthrough `[1 pt]` 
- Install NuGet Package: AspNetCore.HealthChecks.UI

- Add a folder HealthChecks to your solution, and create a class named "AvailableHealthCheck"
- ![][images/healthchecks_1.png]

- In Startup.cs, add the following lines
    - To enable Health Checks in ASP.NET Core, add the following service to the ConfigureServices method:
    ```c#
    services.AddHealthChecks()
        .AddCheck<AvailableHealthCheck>("available", tags: new[] { "ui" });
    ```
    - To enable the User Interface service, add the following:
    ```c
    services.AddHealthChecksUI();
    ```

- In Startup.cs, add the following lines to Configure method:
    ```
    app.UseHealthChecks("/health/api", new HealthCheckOptions()
    {
        Predicate = _ => _.Tags.Contains("api"),
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
    });

- To enable a standard UI for Health Checks, add the following code to Startup.cs, Configure method:
    ```
    app.UseHealthChecksUI();
    ```

### Challenge `[2 pts total]` 
- (200) `[1 pt]` Add a "Random Health Check" to the health checks page
    - How to add your own health check class
- (300) `[1 pt]` Add your own creative Health check, perhaps something Database related?

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

### Challenge

Add a custom pipeline handler
