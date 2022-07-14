# pf-jwt-bearer-dotnet-sample

Demonstrates using PingFederate as the JWT provider for an ASP.Net REST API

## Prerequisites

* ASP.NET Core 6.0
* PingFederate

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

## PingFederate OAuth Client

An OAuth Client with the grant type of Client Credentials is also required:

* Client ID: sampleClient
* Client Name: sampleClient
* Client Authentication: Client Secret
* Client Secret: verySecretPasswordYouPick
* Allowed Grant Types: Client Credentials
* Default Access Token Manager: sampleJWT

## Get a Token

```
curl --location --request POST 'https://auth.example.com/as/token.oauth2' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=sampleClient' \
--data-urlencode 'client_secret=verySecretPasswordYouPick' \
--data-urlencode 'response_type=token' \
--data-urlencode 'grant_type=client_credentials'
```
