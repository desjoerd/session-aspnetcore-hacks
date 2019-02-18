# Session aspnetcore hacks

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
    ```
- To enable a standard UI for Health Checks, add the following code to Startup.cs, Configure method:
    ```
    app.UseHealthChecksUI();
    ```
### Challenge

## JWS Signed Data (200)

### Challenge

## NodeServices (300)

### Challenge

## Mediatr (300)

### Challenge
