---
title: Spine OpenID Connect Implementation
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_spine_implementation.html
summary: An overview of the Spine implementation of OpenID Connect.
---

{% include warning.html content="Work in Progress." %}

## Introduction

The previous pages in this section have provided an overview of OpenID Connect and described the Authorization Code Flow in more detail. The purpose of this page is to describe the current Spine implementation of OpenID Connect.

As an initial step toward offering a fully compliant OpenID Connect solution offering multiple ways to authenticate an End-User, NHS Digital have created an Alpha implementation. This implementation supports a cut down version of OpenID Connect specifically tailored to support native applications deployed on a Windows laptop authenticating users with a Spine smartcard.

This Alpha implementation is described in more detail in the following sections:

## High Level Architecture

The diagram below depicts at a a high level the components involved Authorization Code Flow.

![Spine OpenID Connect Implementation](/images/OIDCSpineImplementation.jpg)

### Components

The following components are involved in the solution:

* **End-Users Laptop**

  The Alpha implementation is specifically tailored to support third party native applications running on a Windows laptop. To support the solution the laptop must have a smartcard reader and have the Spine Identity Agent installed. The laptop must have N3 connectivity. 
  
* **Identity Agent**

  The Identity Agent is a Windows application that controls the authentication of an End-User by requiring them to enter their smartcard into a reader and enter a passcode, this is done in conjunction with the Spine Authentication Server. Currently this is the only authentication mechanism supported by the Spine.
    
  More information about the Identity Agent and how to install it can be found [here](http://nww.hscic.gov.uk/dir/downloads/).
  
* **Identity Agent Bridge**

  The Identity Agent and Spine Authentication Server present a number of APIs that a third party may use to obtain information about an authentication event and the End-User. These APIs are highly bespoke and require third parties to use a range of technologies (e.g. Applets, Java, DLLs, LDAP etc.), typically suppliers have not found this an easy task.
  
  The Identity Agent Bridge is a new component deployed to the End-User's laptop that performs the integration with the Identity Agent on behalf of third party applications. In the place of the existing APIs it presents an OpenID Connect authorization endpoint that a third party may use to initiate an End-User authentication.

* **OpenID Connect Server**

  The OpenID Connect Server is a new component that allows a third party to complete an OpenID Connect authentication by presenting token, JWKS and configuration endpoints.

* **Third Party Application**

  OpenID Connect may be most commonly used by web applications but the current solution is specifically tailored to support third party applications using a client/server architecture comprising a native windows application implementing presentation logic and a server component implementing business logic.
  
  Support for this particular architecture has been provided to allow the development of mobile applications where the user can continue to work even when the laptop is no longer online. Further information on the offline capabilities of this solution is given in later sections.
  
## High Level Flows

This solution supports both online and offline authentication as described below:

### Online Authentication

Online authentication is provided using an Authorization Code Flow as depicted below and described in detail [here](explore_auth_code_flow).

![Authorization Code Flow](/images/OIDCAuthCodeFlow.jpg)

As shown above to initiate an authentication the third party application needs to make an OpenId Connect authentication request to the authorization endpoint. This would typically be done by causing the End-User's browser to perform an HTTP GET request. However in this instance both the third party application and Identity Agent Bridge are native applications so the communication is achieved using the Windows Custom URI Scheme. This requires that the Identity Agent Bridge registers a custom URI to receive authentication requests and that the third party application registers a custom URI to receive the response.

When the third party directs an authentication request at the authorize endpoint the Identity Agent Bridge will interact with the Identity Agent to determine whether the End-User has already authenticated. If they haven't this will start the authentication process by displaying a pop-up window requesting the user to enter their smartcard and enter their passcode.

![Spine Login](/images/OIDCSpineLogin.jpg)

If the user correctly enters their passcode they will then be requested to select a role from those available to them.

![Spine Role Selection](/images/OIDCSpineRoleSelection.jpg)

If the End-User was already authenticated or was newly authenticated the Identity Agent Bridge will interact with the OpenID Connect Server which in turn will interact with the Spine Authentication Server to generate an authorization code. This authorization code will be returned to the third party application by directing an OpenID Connect authentication response to the third party's custom URI. If they authentication fails then an OpenID Connect error response will be returned to the custom URI.

The third party native application should return the authorization code to its server component which can then use it to complete a standard Authorization Code Flow.

### Offline Authentication

{% include warning.html content="Work in Progress." %}

## Implementation Detail

This section provides more detailed information on the use of the Spine implementation of OpenID Connect.

### Registration

{% include warning.html content="Need proper example URI." %}

Before using the Spine implementation of OpenID Connect a third party will need to register their details. This is a manual step as [Dynamic Registration](explore_other_features#dynamic_registration) is not supported. During this step the following values will be agreed:

|Value|Description|
|-----|-----------|
|client_id|An identifier for the third party application which generated by NHS Digital.|
|client_name|A name for the third party application.|
|redirect_uris|The URI(s) to which the third party application may request authorization codes to be redirected. For this implementation these will be custom URIs e.g. clientapp://connect/authresponse|

### Discovery

The Spine implementation does not support [Discovery](explore_other_features#discovery) but it does expose a URI where the configuration data for the OpenID Provider can be retrieved e.g.

```
https://connect.iam.spine2.ncrs.nhs.uk/.well-known/openid-configuration
```

Submitting a HTTP GET request to this URI will return the following JSON document:

```
{
  "token_endpoint": "https://connect.iam.spine2.ncrs.nhs.uk/token",
  "scopes_supported": ["openid"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "subject_types_supported": ["public"],
  "token_endpoint_auth_methods_supported": ["private_key_jwt"],
  "grant_types_supported": ["authorization_code"],
  "response_types_supported": ["code"],
  "issuer": "https://connect.iam.spine2.ncrs.nhs.uk/token",
  "authorization_endpoint": "nhsidentityagent://connect/authorize",
  "claims_supported": ["sub", "iss", "name", "org", "role", "workgroups", "activities"],
  "request_uri_parameter_supported": false,
  "token_endpoint_auth_signing_alg_values_supported": ["RS256"],
  "jwks_uri": "https://connect.iam.spine2.ncrs.nhs.uk/.well-known/jwks.json"
}
```

Things to note are:

1. The authorization endpoint utilises a Custom URI Scheme.
2. Token and JWKS endpoints are provided but the solution does not support a userinfo endpoint.
3. The only scope supported is the openid scope. The profile, email, address and phone scopes are not supported.
4. Only the Authorization Code Flow is supported.
5. Tokens will be signed using RSA utilising the SHA-256 hashing algorithm.

**Note:** the values given above may vary depending on the environment being used. 

### Key Management

The solution provids a JWKS endpoint as described [here](explore_other_features#key_management). The value taken from the JSON document above is:

```
https://connect.iam.spine2.ncrs.nhs.uk/.well-known/jwks.json
```

Submitting a HTTP GET request to this URI will return the following JSON document:

```
{
  "keys":[
    {"kid":"rF-Do8xhajL-bfrC0Ef0DpqvwbJ9UUzUbEFQwl42rw4",
     "e":"AQAB",
     "kty":"RSA",
     "n":"vsqnkkTNOlMC9boScmBCFXQ2BpQwdOGd3ET55PY4wBzZ217opPqiP43_yZBgWfG3Mb1r3PaDNCtAZSbsWH_iIvLZnllnvzYY
         _iOW7zrqaWSD4Dfaaa2Eg4dOBI71n4A38FRsa2N1nbWKCdX_ffZwqxZwX0dg44ELvwO5rTwJLoIZPXy2FD_lkjrR8ksC_N9EiQ
         -yY9bbQTIgVZuHXdklZnTTCeUczz7bAt3DVJ5JFOT1E7sBkUqOqwXLG7k-capqWYJXUNgKFig0dt8AEPPvAhXsnt4fKOQQx2Kb
         y-YNYijpxP752P62ArGLaqcvv5hU13V2obPMQ4P9ysHBFyEurw"
    }
  ]
}
```

This shows a single RSA key being returned. Periodically NHS Digital will rotate its signing key using a three step process:

1. A period of time prior to using a new key it will be added to the JWK Set with a new identifier.
2. The new key will be used to sign tokens.
3. A period of time after the old key is no longer used it will be removed from the JWK Set.

To avoid having to read the JWKS endpoint every time a signature is checked suppliers are encouraged to locally cache the JWK Set. However they should be aware of the rotation procedure described and re-query the JWKS endpoint when they receive a kid they do not recognise.

**Note:** the values given above may vary depending on the environment being used. 

{% include warning.html content="Need to check how the third party is authenticated and whether they need to provide a JWKS endpoint." %}

### Authentication Request

This is as described for the Authorization Code Flow detailed [here](explore_auth_code_flow#authentication_request) with the exception that the request is made to the Identity Agent custom URI e.g.

```
nhsidentityagent://connect/authorize?scope=openid&response_type=code&redirect_uri=clientapp%3A%2F%2Fconnect%2authresponse&state=af0ifjsldkj
```

The current implementation currently only supports the scope, response_type, redirct_uri and state parameters with the only scope supported being openid.

### Authentication Response

#### ID Token

```
{"role":"S0080:G0450:R5080","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","jti":"0cca2fe5675f4fe28f687ec5a2dac187","activities":["B0005","B0008"],"exp":1503584507}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - Refresh Token Decoded: {"iv":"LopflBFmvq3BJRUm5r2v3cGpVE7iEyLN4Qf8YLvXS4o=","role_id":"240000115894","device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","sso_ticket":"3zkYKhNMgbFwVqAy0aBS8W93epK8k4xsBWP-8sbns0SJmr1z2B5hFJHbe8txQQ2eluxmULLB2ogxHYpZvyQ=","jti":"2617f03c0b44454b93b427c5d77e704d","tag":"dkQryHQjQ1KrYlFDaNz_3Q==","exp":1503620477}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - ID Token Decoded: {"device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","role":{"role_id":"240000115894","name":"\"Admin & Clerical\":\"Management - A & C\":\"Registration Authority Manager\"","code":"S0080:G0450:R5080"},"activities":["B0005","B0008"],"amr":["pin","legacy_signed_challenge"],"jti":"2b44cf3602d445e0909a808ad3194341","name":"Seven User Mr","s_hash":"LLJ6GrrF3GenakaKAyYBTg","at_hash":"gNn8ijUTnwM34MxSFzBgYw","sub":"uid=240000109896,ou=People,o=nhs","org":{"name":"GREATER MANCHESTER STRATEGIC HA","code":"Q14"},"iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"s6BhdRkqt3","exp":1503620477,"c_hash":"MPZJ1jWGHrGqL-fMHAEBMQ"}
```

