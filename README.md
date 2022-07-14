
Demonstrates using PingFederate as the JWT provider for an ASP.Net API.  This sample code uses Microsoft.AspNetCore.Authentication.JwtBearer

## Prerequisites

* ASP.NET Core 6.0
* PingFederate
* Trusted SSL Certificate
* curl
* dotnet cli

## Assumptions

Your PingFederate runtime is available at the URL https://pingfederate:9031.  You are running this sample dotnet application locally, and it will be available at http://localhost:5000.  Update accordingly.

A valid SSL certificate is assigned to your PingFederate instance.  The dotnet application will fail to validate your JWT without a signed certificate.

## PingFederate Access Token Manager

We need an Access Token Manager of type JSON Web Token in PingFederate.  Default values are great, with a couple of exceptions:

* Type - Instance Name: sampleJWT
* Type - Instance ID: sampleJWT
* Type - Type: JSON Web Tokens
* Instance Configuration - Use Centralized Signing Key: Enabled
* Instance Configuration - JWS Algorithm: RSA using SHA-256
* Instance Configuration (Advanced) - Issuer Claim Value: https://pingfederate:9031
* Instance Configuration (Advanced) - Audience Claim Value: http://localhost:5000
* Access Token Attribute Contract - Extend the Contract: sub

## PingFederate OAuth Client

An OAuth Client with the grant type of Client Credentials is also required:

* Client ID: sampleClient
* Client Name: sampleClient
* Client Authentication: Client Secret
* Client Secret: verySecretPasswordYouPick
* Allowed Grant Types: Client Credentials
* Default Access Token Manager: sampleJWT

## Clone this repository

```Shell
git clone https://github.com/mdeller-ping/pf-jwt-bearer-dotnet-sample.git
cd pf-jwt-bearer-dotnet-sample/pf-jwt-bearer-dotnet-sample
```

## Edit Program.cs

The Issuer (aka Authority) and Audience values are hard coded in Program.cs.  Update these values with your real URLs

```C#
.AddJwtBearer(options =>
{
    options.Authority = "https://pingfederate:9031";
    options.Audience = "http://localhost:5000";
});
```

## Run the sample

```Shell
dotnet run
```

## Get a Token

In a different terminal window or command prompt:

```cURL
curl --location --request POST 'https://pingfederate:9031/as/token.oauth2' \
  --insecure \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'client_id=sampleClient' \
  --data-urlencode 'client_secret=verySecretPasswordYouPick' \
  --data-urlencode 'response_type=token' \
  --data-urlencode 'grant_type=client_credentials'
```
## Use your Token

```cURL
curl --location --request GET 'http://localhost:5000/WeatherForecast' \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_JWT_FROM_PRIOR_STEP' \
  --insecure \
  --verbose
```

## Is this repo required for JWT validation with PingFederate?

No.  The Microsoft.AspNetCore.Authentication.JwtBearer library is doing the heavy lifting.  That library is available as a NuGet package.  To use this library in a your own project:

### Create a Project in Visual Studio 2022

ASP.NET Core - API

### Add Package Dependency

Use NuGet to add the package Microsoft.AspNetCore.Authentication.JwtBearer

### Modify Program.cs

ASP.NET 6.0 eliminated Startup.cs.  Instead we will be working in Program.cs.

Include the JwtBearer library

```C#
using Microsoft.AspNetCore.Authentication.JwtBearer;
```

Specify the authentication mechanism

```C#
builder.Services.AddAuthentication(opt => {
    opt.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    opt.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.Authority = "https://pingfederate:9031";
    options.Audience = "http://localhost:5000";
});
```

Enable Authentication

```C#
app.UseAuthentication();
```

The completed Program.cs

```C#
using Microsoft.AspNetCore.Authentication.JwtBearer;

var builder = WebApplication.CreateBuilder(args);

// Begin JSON Web Token Stuff

builder.Services.AddAuthentication(opt => {
    opt.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    opt.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.Authority = "https://pingfederate:9031";
    options.Audience = "http://localhost:5000";
});

// End JSON Web Token Stuff

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Modify Controllers/WeatherForecastControllers.cs

Include the Authorization library

```C#
using Microsoft.AspNetCore.Authorization;
```

Change the following line:

```C#
    [HttpGet(Name = "GetWeatherForecast")]
```

We need to include Authorize:

```C#
    [HttpGet(Name = "GetWeatherForecast"), Authorize]
```

The completed Controllers/WeatherForecastController.cs

```C#
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;

namespace pf_jwt_bearer_dotnet_sample.Controllers;

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet(Name = "GetWeatherForecast"), Authorize]
    public IEnumerable<WeatherForecast> Get()
    {
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        })
        .ToArray();
    }
}
```
