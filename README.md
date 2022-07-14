# pf-jwt-bearer-dotnet-sample

Demonstrates using PingFederate as the JWT provider for an ASP.Net REST API

## Prerequisites

* ASP.NET Core 6.0
* PingFederate

## PingFederate Access Token Manager

We need an Access Token Manager of type JSON Web Token in PingFederate.  Default values are great, with a couple of exceptions:

* Type - Instance Name: jsonWebToken
* Type - Instance ID: jsonWebToken
* Type - Type: JSON Web Tokens
* Instance Configuration - Use Centralized Signing Key: Enabled
* Instance Configuration - JWS Algorithm: RSA using SHA-256
* Instance Configuration (Advanced) - Issuer Claim Value: Your PingFederate Runtime URL (e.g., https://auth.example.com:9031)
* Instance Configuration (Advanced) - Audience Claim Value: Your Application URL (e.g., https://api.example.com)

## PingFederate OAuth Client

An OAuth Client with the grant type of Client Credentials is also required:

* Client ID: client_credentials
* Client Name: client_credentials
* Client Authentication: Client Secret
* Client Secret: 2FederateM0re
* Allowed Grant Types: Client Credentials
* Default Access Token Manager: jsonWebToken

