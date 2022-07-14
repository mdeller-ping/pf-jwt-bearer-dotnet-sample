# pf-jwt-bearer-dotnet-sample

Demonstrates using PingFederate as the JWT provider for an ASP.Net API.  This sample code uses Microsoft.AspNetCore.Authentication.JwtBearer

## Prerequisites

* ASP.NET Core 6.0
* PingFederate
* curl
* dotnet cli

## Assumptions

Your PingFederate runtime is available at the URL https://auth.example.com.  You are running this sample dotnet application locally, and it will be available at http://localhost:5000.  Update accordingly.

## PingFederate Access Token Manager

We need an Access Token Manager of type JSON Web Token in PingFederate.  Default values are great, with a couple of exceptions:

* Type - Instance Name: sampleJWT
* Type - Instance ID: sampleJWT
* Type - Type: JSON Web Tokens
* Instance Configuration - Use Centralized Signing Key: Enabled
* Instance Configuration - JWS Algorithm: RSA using SHA-256
* Instance Configuration (Advanced) - Issuer Claim Value: https://auth.example.com
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

```
git clone https://github.com/mdeller-ping/pf-jwt-bearer-dotnet-sample.git
```

## Edit pf-jwt-bearer-dotnet-sample/Program.cs

The Issuer (aka Authority) and Audience values are hard coded in pf-jwt-bearer-dotnet-sample/Program.cs.  Update these values with your real URLs

```
.AddJwtBearer(options =>
{
    options.Authority = "https://auth.example.com";
    options.Audience = "http://localhost:5000";
});
```

## Run the sample

Change to the directory with pf-jwt-bearer-dotnet-sample.csproj and run the project

```
dotnet run
```

## Get a Token

In a different terminal window or command prompt:

```
curl --location --request POST 'https://auth.example.com/as/token.oauth2' \
  --insecure \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'client_id=sampleClient' \
  --data-urlencode 'client_secret=verySecretPasswordYouPick' \
  --data-urlencode 'response_type=token' \
  --data-urlencode 'grant_type=client_credentials'
```
## Use your Token

```
curl -H 'Accept: application/json' \
  --insecure \
  --verbose \
  -H "Authorization: Bearer YOUR_JWT_FROM_PRIOR_STEP" \
  http://localhost:5000/WeatherForecast
```
