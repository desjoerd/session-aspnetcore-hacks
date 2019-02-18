# Session aspnetcore hacks

For this workshop we've got a couple of hacks for AspNetCore. Each hack has a detailed walkthrough and a challenge. You can choose which one you want to do.
The start project is always the one provided in this repository which is a *default aspnetcore empty* app with `swashbuckle.aspnetcore` added.

## Healthchecks (200)
HealthChecks in ASP.NET Core are ......... [link]

### Walkthrough
- Install NuGet Package: AspNetCore.HealthChecks.UI

- In Startup.cs, add the following lines to ConfigureServices method:
    ```csharp
    services.AddHealthChecks()
        .AddCheck<RandomHealthCheck>("random", tags: new[] { "api", "ui" })
        .AddCheck<AvailableHealthCheck>("available", tags: new[] { "ui" });
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

### Challenge

## JWS Signed Data (200)

JWS stands for Json Web Signature and is distrubuted as a Json Web Token (JWT).
An json webtoken consists of three parts which are `base64` encoded seperated by an `.`.

- The first part is the header which contains the type of token and the signature/encryption algorithm.
- The second part is the actual `json` payload which can be encrypted.
- The last part is the signature

https://jwt.io contains a debugger for json web tokens.
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
For this workshop we are going to use JWS tokens which is a Json Web Token with a signature. Almost every JWT used for authentication is an JWS, but can also be used to share data between services. Because of the signature the JWS can be used to share data between two (micro)services via an untrusted source (ie an Javascript web client) and cannot be tampered.

### Walktrough


### Challenge

## NodeServices (300)

### Challenge

## Mediatr (300)

### Challenge
