---
title: Other OpenID Connect Features
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_other_features.html
summary: An overview of other features of the OpenID Connect specifications.
---

## Introduction

The previous pages introduced the high level concepts of the OpenID Connect Core Specification and provided a more detailed explanation of the Authorization Code Flow.

The purpose of this page is to give an overview of some of the aspects of the wider set of OpenID Connect specifications (the full set of which can be found [here](http://openid.net/connect/)).  

## Discovery

In order to be able to interact with an OpenID Provider to authenticate an End-User a Relying Party needs to know how to contact the provider and the features that it supports.

This information may be provided to the Relying Party as a manual step during the registration process but OpenID Connect contains a standard ([OpenID Connect Discovery 1.0](http://openid.net/specs/openid-connect-discovery-1_0.html)) that allows this to be electronically done via the [WebFinger](https://tools.ietf.org/html/rfc7033) protocol.

To do this the Relying Party needs to know an identifier for the End-User they wish to authenticate and the location of a WebFinger service. It can then make a request to the WebFinger server as in the example below:

```
 GET /.well-known/webfinger
    ?resource=acct%3Ajoe%40example.com
    &rel=http%3A%2F%2Fopenid.net%2Fspecs%2Fconnect%2F1.0%2Fissuer
    HTTP/1.1
  Host: example.com
```

This will result in a reference to the OpenID Provider host being returned as in the example below:

```
  HTTP/1.1 200 OK
  Content-Type: application/jrd+json

  {
   "subject": "acct:joe@example.com",
   "links":
    [
     {
      "rel": "http://openid.net/specs/connect/1.0/issuer",
      "href": "https://server.example.com"
     }
    ]
  } 
```

Once the Relying Party has located the OpenID Provider it can obtain details about its configuration by submitting a request for the the [well-known](https://tools.ietf.org/html/rfc5785) openid-configuration page as in the example below.

```
  GET /.well-known/openid-configuration HTTP/1.1
  Host: example.com
```

This will result in a series of claims about the OpenID Provider's configuration being returned as in the example below:

```
HTTP/1.1 200 OK
  Content-Type: application/json

  {
   "issuer":
     "https://server.example.com",
   "authorization_endpoint":
     "https://server.example.com/connect/authorize",
   "token_endpoint":
     "https://server.example.com/connect/token",
   "token_endpoint_auth_methods_supported":
     ["client_secret_basic", "private_key_jwt"],
   "token_endpoint_auth_signing_alg_values_supported":
     ["RS256", "ES256"],
   "userinfo_endpoint":
     "https://server.example.com/connect/userinfo",
   "jwks_uri":
     "https://server.example.com/jwks.json",
   "registration_endpoint":
     "https://server.example.com/connect/register",
   "scopes_supported":
     ["openid", "profile", "email", "address",
      "phone", "offline_access"],
   "userinfo_signing_alg_values_supported":
     ["RS256", "ES256", "HS256"],
   "userinfo_encryption_alg_values_supported":
     ["RSA1_5", "A128KW"],
   "id_token_signing_alg_values_supported":
     ["RS256", "ES256", "HS256"],
   "display_values_supported":
     ["page", "popup"],
   "service_documentation":
     "http://server.example.com/connect/service_documentation.html",
  }
```
The full set of OpenID Provider metadata can be found in the [OpenID Connect Discovery Specification](http://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata).

Note that an OpenID Provider may support a well-known openid-configuration page even if they do not support discovery by a WebFinger server.

## Dynamic Registration

In order to use the services offered by an OpenID Provider a Relying Party must register with the provider. During this step a client identifier will be assigned to the Relying Party and metadata about its configuration recorded.

This may be done manually but OpenID Connect contains a specification ([OpenID Connect Dynamic Client Registration 1.0](http://openid.net/specs/openid-connect-registration-1_0.html)) that allows this to be electronically.

OpenID Providers that support this specification provide a client registration endpoint. A client requests registration by sending a HTTP POST request as in the example below. The OpenID Provider may require an initial access token that is provisioned out-of-band (as shown in the example) to restrict registration requests to only authorized clients or developers.

```
POST /connect/register HTTP/1.1
  Content-Type: application/json
  Accept: application/json
  Host: server.example.com
  Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJ ...

  {
   "application_type": "web",
   "redirect_uris":
     ["https://client.example.org/callback",
      "https://client.example.org/callback2"],
   "client_name": "My Example",
   "logo_uri": "https://client.example.org/logo.png",
   "token_endpoint_auth_method": "client_secret_basic",
   "jwks_uri": "https://client.example.org/my_public_keys.jwks",
   "userinfo_encrypted_response_alg": "RSA1_5",
   "userinfo_encrypted_response_enc": "A128CBC-HS256",
   "contacts": ["ve7jtb@example.org", "mary@example.org"],
  }
```

The OpenID Provider will assign the Relying Party a unique client identifier, optionally assign a client secret, and associate the metadata provided with the issued client identifier. The OpenID Provider may provision default values for any items omitted in the client metadata.

A HTTP 201 Created response will be returned to the Relying Party as in the example below:

```
HTTP/1.1 201 Created
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "client_id": "s6BhdRkqt3",
   "client_secret":
     "ZJYCqe3GGRvdrudKyZS0XhGv_Z45DuKhCUk0gBR1vZk",
   "client_secret_expires_at": 1577858400,
   "registration_access_token":
     "this.is.an.access.token.value.ffx83",
   "registration_client_uri":
     "https://server.example.com/connect/register?client_id=s6BhdRkqt3",
   "token_endpoint_auth_method":
     "client_secret_basic",
   "application_type": "web",
   "redirect_uris":
     ["https://client.example.org/callback",
      "https://client.example.org/callback2"],
   "client_name": "My Example",
   "jwks_uri": "https://client.example.org/my_public_keys.jwks",
   "userinfo_encrypted_response_alg": "RSA1_5",
   "userinfo_encrypted_response_enc": "A128CBC-HS256",
   "contacts": ["ve7jtb@example.org", "mary@example.org"],
  }
```

The response will include the newly-created client identifier and, if applicable, a Client Secret, along with all registered metadata about this Client, including any fields provisioned by the OpenID Provider itself.

Full details of the metadata that can be registered can be found in the [OpenID Connect Dynamic Client Registration 1.0 Specification](http://openid.net/specs/openid-connect-registration-1_0.html) together with details of how an OpenID Provider may optionally provide a  client configuration endpoint allowing a Relying Party to view and update its registered metadata.

## Key Management

Asymmetric keys are used in a number of ways in OpenID Connect:

* The Relying Party may use a JWT signed with a private key to authenticate to the OpenID Provider when forming a Token Request as described in [here](explore_auth_code_flow#client-authentication).
* The OpenID Provider may respond to a request with a JWT either signed or encrypted with a private key as described [here](explore_auth_code_flow#token-successful-response)

The header of the signed JWT identifies the algorithm used to sign the JSON object and the identity of the key used as shown in the example below:

```
{"alg":"RS256","kid":"1e9gdk7"}
```

A Relying Party or OpenID Provider party attempting to verify a signature needs to obtain the public key corresponding to the kid provided. This data could be exchanged manually at the time of registration but this does readily not allow for signing keys to be rotated. 

To address this issue signers publish their keys in a JSON Web Key ([JWK](https://tools.ietf.org/html/rfc7517)) Set at a published location, defined by the jkws_uri configuration item (see above), and include the kid of the signing key in the header of each JWT to indicate which key is to be used to validate the signature. 

Keys can then be rolled over by periodically adding new keys to the JWK Set. The signer can begin using a new key at its discretion and signals the change to the verifier using the kid value. The verifier knows to go back to the jwks_uri location to re-retrieve the keys when it sees an unfamiliar kid value. The JWK Set document at the jwks_uri will retain recently decommissioned signing keys for a reasonable period of time to facilitate a smooth transition.

A party's JWK Set can be retrieved by sending a HTTP GET request to the jwks_uri location as in the example below:

```
  GET /jwks HTTP/1.1
  Host: server.example.com
```

This will result in a JSON document being returned. In the example below a single key with the kid 1e9gdk7 has been returned. It is an RSA key with the modulus given by n and the public exponent by e. 

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

  {
   "keys":[
   {
    "kty":"RSA",
    "kid":"1e9gdk7",
    "n":"w7Zdfmece8iaB0kiTY8pCtiBtzbptJmP28nSWwtdjRu0f2GFpajvWE4VhfJA
         jEsOcwYzay7XGN0b-X84BfC8hmCTOj2b2eHT7NsZegFPKRUQzJ9wW8ipn_aD
         JWMGDuB1XyqT1E7DYqjUCEOD1b4FLpy_xPn6oV_TYOfQ9fZdbE5HGxJUzeku
         GcOKqOQ8M7wfYHhHHLxGpQVgL0apWuP2gDDOdTtpuld4D2LK1MZK99s9gaSj
         RHE8JDb1Z4IGhEcEyzkxswVdPndUWzfvWBBWXWxtSUvQGBRkuy1BHOa4sP6F
         KjWEeeF7gm7UMs2Nm2QUgNZw6xvEDGaLk4KASdIxRQ",
    "e":"AQAB"
    }
   ]
  }
```

For further details see the [JWK Specification](https://tools.ietf.org/html/rfc7517).

## Session Management

## Front-Channel Logout

## Back-Channel Logout
