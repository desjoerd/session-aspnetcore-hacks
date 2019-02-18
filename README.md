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

### Challenge

## NodeServices (300)

### Challenge

## Mediatr (300)

### Challenge
