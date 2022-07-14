# pf-jwt-bearer-dotnet-sample

Demonstrates using PingFederate as the JWT provider for an ASP.Net API.  This sample code uses Microsoft.AspNetCore.Authentication.JwtBearer

## Prerequisites

* ASP.NET Core 6.0
* PingFederate
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

```
git clone https://github.com/mdeller-ping/pf-jwt-bearer-dotnet-sample.git
cd pf-jwt-bearer-dotnet-sample/pf-jwt-bearer-dotnet-sample
```

## Edit Program.cs

The Issuer (aka Authority) and Audience values are hard coded in Program.cs.  Update these values with your real URLs

```
.AddJwtBearer(options =>
{
    options.Authority = "https://pingfederate:9031";
    options.Audience = "http://localhost:5000";
});
```

## Run the sample

```
dotnet run
```

## Get a Token

In a different terminal window or command prompt:

```
curl --location --request POST 'https://pingfederate:9031/as/token.oauth2' \
  --insecure \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'client_id=sampleClient' \
  --data-urlencode 'client_secret=verySecretPasswordYouPick' \
  --data-urlencode 'response_type=token' \
  --data-urlencode 'grant_type=client_credentials'
```
## Use your Token

```
curl --location --request GET 'http://localhost:5000/WeatherForecast' \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_JWT_FROM_PRIOR_STEP' \
  --insecure \
  --verbose
```
