---
title: The Authorization Code Flow in Detail
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_auth_code_flow.html
summary: A detailed description of the Authorization Code Flow.
---

## Introduction

The Authorization Code Flow is the most commonly used variant of the OpenID Connect authentication flows. It is suited for use with web applications and native applications that utilise a client/server architecture. In this flow rather than return the id, access and refresh tokens directly to the Relying Party's client component an authorization code is returned. The Relying Party's server component can then exchange the code for the required tokens.

This provides the dual benefits of:

1. Not exposing any tokens to the user agent or applications with access to the user agent.
2. Allowing the Relying Party to be authenticated before exchanging the code for tokens.

This flow requires that a Relying Party can securely maintain a client secret between themselves and the OpenID Provider.

## High Level Flow

The diagram below depicts at a a high level the Authorization Code Flow.

![Authorization Code Flow](images/OIDCAuthCodeFlow.jpg)

1. The Relying Party sends a request to the OpenID Provider to authenticate the End-User. The request must include the Relying Party's identity and the openid scope, it may optionally include other scopes e.g. the email scope if the Relying Party wishes to obtain the user's email address.
2. The OpenID Provider authenticates the End-User using one of the methods available to it and obtains authorization from the End-user to provide the requested scopes to the identified Relying Party.
3. Once the End-User has been authenticated and has authorized the request the OpenID Provider will return an authorization code to the Relying Party's server component.
4. The Relying Party's server component contacts the token endpoint and exchanges the authorization code for an id token identifying the End-User and optionally access and refresh tokens granting access to the userinfo endpoint.
4. Optionally the Relying Party may request the additional user information (e.g. email address) from the userInfo endpoint by presenting the access token obtained in the previous step.

This flow is described in much more detail in the following sections.

## Registration

Before being able to interact with an OpenID Provider a Relying Party must first register with the provider. During this process the following items will be exchanged:

1. The URIs of the OpenID Provider's authorization, token and user endpoints.
2. An issuer identifier identifying the OpenID Provider.
3. A mechanism for obtaining the OpenID Provider's public keys.
4. A client identifier uniquely identifying the Relying Party.
3. The Relying Party's redirection URIs to which responses may be redirected.
5. A client authentication mechanism and associated credentials.

This may be achieved via manual means. However OpenID Connect also provides an electonic way of doing this, see the [Discovery](explore_other_features#discovery) and [Dynamic Registration](explore_other_features#dynamic_registration) sections in the the Other OpenID Connect Features page for further details.

## Authentication Request

When a Relying Party requires that an End-User is authenticated they should cause a HTTP GET request over TLS to be sent from the End-User's user agent to the OpenID Provider's authorization endpoint.

The request may be generated indirectly via a HTTP 302 redirect response (for example in response to a user attempting to access a protected resource) or directly e.g. as a result of a login button being hit. An example of each is given below:

```
HTTP/1.1 302 Found
  Location: https://openidprovider.example.com/authorize?
    response_type=code
    &scope=openid%20profile%20email
    &client_id=s6BhdRkqt3
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

```
GET /authorize?
    response_type=code
    &scope=openid%20profile%20email
    &client_id=s6BhdRkqt3
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
```
The OpenID Connect Core Specification defines a number of mandatory and recommended parameters to use in the request. The minimum recommended set is as follows:

|Name|Description|
|----|-----------|
|scope|This is a space delimited list of the scopes requested by the client. It must contain the value openid and may contain other values e.g. profile and email. Unrecognised values will be ignored.|
|response_type|This defines the processing flow to be used when forming the response. When using the Authorization Code Flow, this value must be code.|
|client_id|This must contain the client identifier assigned to the Relying Party during its registration with the OpenID Provider.|
|redirect_uri|This is the URI to which the response should be sent. This must exactly match one of the Relying Party's redirection URIs registered with the OpenID Provider.|
|state|It is recommended that client's use this parameter to maintain state between the request and the callback. Typically, Cross-Site Request Forgery (CSRF, XSRF) mitigation is done by cryptographically binding the value of this parameter with a browser cookie.|

The OpenID Connect Core Specification also defines a number of optional parameters that may be used to modify the behaviour of the authentication process. For example the prompt parameter can be used to control whether the user is prompted for re-authentication or not. 

For more details see the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth). It should be noted that support for optional parameters is implementation specific.

### Authentication and Authorization

If the request is valid the OpenID Provider will attempt to authenticate the End-User. The means used to do this are outside the scope of the OpenID Connect Core Specification and will vary by implementation, but typically this will be done by loading a page in the End-User's user agent requesting the user to provide their credentials e.g. username and password and possibly a second factor like a One Time Password.

Once the End-User is authenticated the OpenID Provider must obtain an authorization decision before releasing information to the Relying Party. This may be done through an interactive dialogue with the End-User that makes it clear what is being consented to or by other means (for example, via previous administrative consent).

The diagram below depicts example screen screens that might be implemented by an OpenID Provider:

![Authentication and Authorization Screens](images/OIDCLoginScreens.jpg)

The manner in which the authentication and authorization are performed can be controlled to some degree by optional parameters in the authentication request as follows:

|Name|Description|
|----|-----------|
|display|A string value that specifies how the the authentication and consent user interface pages are displayed to the End-User e.g. the value popup will cause the user agent to use a popup window.|
|prompt|A list of string values that specifies whether to prompt the End-User for re-authentication and consent e.g. the value login will prompt the user for re-authentication even if they are already authenticated.|
|max_age|Specifies the allowable elapsed time since the last time the End-User was actively authenticated. If the elapsed time is exceeded the End-User will be re-authenticated.|
|ui_locales|The End-User's preferred languages and scripts for the user interface.|
|id_token_hint|Specifies an id token previously issued by the OpenID Provider Authorization. If the End-User identified by the token is already logged in or is logged in by the request, then a positive response will be returned; otherwise an error will be returned.|
|login_hint|A hint about the login identifier the End-User might use to log in. For example this hint can be used by a Relying Party if it first asks the End-User for their e-mail address (or other identifier) and then wants to pass that value as a hint to the OpenID Provider.|
|acr_values|A list of string values specifying the authentication contexts to be used by the OpenID Provider in order of preference. This effectively allows the Relying Party to specify the strength of authentication performed e.g. Strong 2 Factor. The use of this parameter and the acr values supported is implementation specific.|

Full details of these parameters can be found in the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth). It should be noted that support for these parameters and the behaviour they drive is implementation specific.

### Authentication Successful Response

If the End-User is successfully authenticated and authorizes access to the data requested the OpenID Provider will return an authorisation code to the Relying Party's server component. This is achieved by returning a HTTP 302 redirect request to the End User's user agent requesting that the response is redirected to the redirection_uri specified in the authorize request.

An example response is given below:

```
  HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

The response will contain a code parameter and optionally a state parameter. 

The code parameter holds the authorization code which is a string value. Some OpenID Provider implementations may encode state about the id token to be returned in the authorization code value, whilst others may use the authorization code value as an index into a data sore holding this state. In either case the content of authorization code is opaque to the Relying Party.

The state parameter will be returned if a value was provided by the Relying Party in the authorize request. The Relying Party should validate that the value returned matches that supplied. The state value can additionally be used to mitigate against XSRF attacks by cryptographically binding the value of this parameter with a browser cookie (for more details see the [OAuth 2.0 Authorization Framework Specification](https://tools.ietf.org/html/rfc6749#section-10.12)).

### Authentication Error Response

If the End-User denies the request or the authentication fails the OpenID Provider will return an error response to the Relying Party. As for a successful response this is achieved by returning a HTTP 302 redirect request to the End User's user agent requesting that the response is redirected to the redirection_uri specified in the authorize request.

If the redirection URI specified is invalid an error will be returned directly to the user agent.

HTTP errors unrelated to OpenID Connect will be returned to the user agent using the appropriate HTTP status code.

An example error response is given below:

```
  HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
    &state=af0ifjsldkj
```

The response will contain an error parameter and optionally error_description, error_uri and state parameters. The error_uri parameter may be used by implementations to specify a human-readable web page with information about the error, used to provide the client developer with additional information about the error.

Standard error values are defined in the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequestValidation). Implementations may also define their own errors.

## Token Request

Once the Relying Party server component has received an authorization code it can request the OpenID Provider to provide the associated id, access and refresh tokens. It does this by sending a HTTP POST request over TLS directly to the OpenID Provider's token endpoint URI as per the example below:

```
  POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

The following serialized form parameters must be provided in the request:

|Name|Description|
|----|-----------|
|grant_type|This must be set to authorization_code.|
|code|The authorization code received in response to the authentication request.|
|redirect_uri|The redirection URI supplied in the original authentication request.|
|client_id|The Relying Party client identifier (not required for HTTP Basic Authentication).| 


### Client Authentication

As part of the registration process the Relying Party and OpenID Provider will have agreed a mechanism by which the Relying Party can be authenticated when making the token request. This is required to ensure that the request is genuine and that the tokens are not returned to a third party masquerading as the Relying Party. 

The authentication mechanisms available will depend on the implementation but the OpenID Connect Core Specification supports the following methods:

|Method|Description|
|------|-----------|
|client_secret_basic|Authentication using a client secret value in conjunction with the HTTP Basic authentication scheme. This is the default method and is illustrated in the example above.|
|client_secret_post|Authentication using a client secret value included in a client_secret parameter in the request body.|
|client_secret_jwt|Authentication using a JWT created with a Hash-based Message Authentication Code (HMAC) calculated from a client secret used as a shared key. The JWT must be sent as the value of a client_assertion parameter with a client_assertion_type parameter set to urn:ietf:params:oauth:client-assertion-type:jwt-bearer.|
|private_key_jwt|Authentication using a JWT signed with a public key registered to the Relying Party. The JWT must be sent as the value of a client_assertion parameter with a client_assertion_type parameter set to urn:ietf:params:oauth:client-assertion-type:jwt-bearer.|
|none|No authentication performed.|

The example below shows the use of a signed JWT to authenticate (in the example JWT value has been abbreviated).

```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded

  grant_type=authorization_code&
  code=i1WsRn1uB1&
  redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb&
  client_id=s6BhdRkqt3&
  client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
  client_assertion=eyJhbGciOiJFUzI1NiIsImtpZCI6IjE2In0.a ... b.c ... d
```

The client assertion JWT is comprised of three Base64URL encoded elements separated by a . character. The first element is the token header. If we decode the value from the example above we get the string below: 

```
{"alg":"ES256","kid":"16"}
```

This specifies that the token has been signed with ECDSA utilising a P-256 curve and SHA-256 hash algorithm using the key identified by the string "16". Details on how the Relying Party may obtain the public key associated with kid are given in the [Key Management](explore_other_features#key_management) section in the the Other OpenID Connect Features page.

Decoding the second element would give the JSON object containing the claims about the Relying Party e.g. the client identifier, expiration date etc.

Full details on client authentication can be found in the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). 
 
### Token Request Validation

On receiving the request the OpenID Provider will authenticate the Relying Party, validate that the parameters provided in the token request are consistent with those in the authentication request and check that the authorisation code was issued to the authenticated Relying Party. 

The OpenID Provider may also perform additional security checks, validating that the authorization code has not been used before and has not expired (typically an authorisation code is given short lifetime).

### Token Successful Response

If the request validation is successful the OpenID Provider will return an HTTP 200 OK response including id, access and refresh tokens as in the example below:

*Headers*

```
Header Name	Header Value
Cache-Control	no-store
Pragma	no-cache
```

*Body*

```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "access_token": "SlAV32hkKG",
   "token_type": "Bearer",
   "refresh_token": "8xLOxBtZp8",
   "expires_in": 3600,
   "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
     yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
     NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
     fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
     AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
     Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
     NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
     QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
     K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
     XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
  }
```

The response body will include the following parameters:

|Name|Description|
|----|-----------|
|acess_token|The access token which may be used to access the userinfo endpoint.|
|token_type|Set to Bearer.|
|refresh_token|A refresh token which can be used to obtain a new access token.|
|expires_in|The lifetime in seconds of the access token.|
|id_token|The Base64URL encoded id token.|

The id token is comprised of three Base64URL encoded elements separated by a . character. The first element is the id token header. If we decode the value from the example above we get the string below: 

```
{"alg":"RS256","kid":"1e9gdk7"}
```

This specifies that the token has been signed with an RSA Signature utilising the SHA-256 hashing algorithm and the key identified by the string "1e9gdk7". Details on how the Relying Party may obtain the public key associated with kid are given in the [Key Management](explore_other_features#key_management) section in the the Other OpenID Connect Features page.

Decoding the second element gives us the JSON object containing the claims about the user. For example decoding the value from the example above gives:

```
{
 "iss": "http://server.example.com",
 "sub": "248289761001",
 "aud": "s6BhdRkqt3",
 "nonce": "n-0S6_WzA2Mj",
 "exp": 1311281970,
 "iat": 1311280970
}
```

The third element is the signature over the JSON object. Details on how this signature is created and on how to validate it can be found in the [JSON Web Signature Specification](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41).

#### ID Token Validation

The [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation) specifies the validation the Relying Party should perform on the id token. This includes but is not limited to the the following:

1. The issuer identifier for the OpenID Provider must exactly match the value of the iss claim.
2. The aud claim must contain the client id value of the Relying Party.
3. The OpenID Provider must be authenticated. The Relying Party may choose to do this by relying on TLS server validation or by checking the token signature.
4. The current time must be before the time represented by the exp claim.

#### Access Token Validation

An OpenID Provider may optionally include an additional at_hash claim in the id token. It will hold a Base64URL encoding of the left-most half of the hash of the access token value, where the hash algorithm used is specified in the alg parameter of the id token header. The Relying Party can use this value to check the integrity of the access token.

### Token Error Response
 
If the Token Request is invalid or unauthorized an HTTP 400 response will be returned as in the following example:

```
  HTTP/1.1 400 Bad Request
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "error": "invalid_request"
  }
```
The response will contain an error parameter and optionally error_description and error_uri parameters. The error_uri parameter may be used by implementations to specify a human-readable web page with information about the error, used to provide the client developer with additional information about the error.

Standard error values are defined in the [OAuth 2.0 Authorization Framework Specification](https://tools.ietf.org/html/rfc6749#section-5.2). Implementations may also define their own errors.

## UserInfo Request

Typically the id token only contains claims about the authentication event and the identity of the End-User. Other information about the End-User can be requested by including additional scopes in the authentication request as in the [example request](#authentication-request) above which includes the standard profile and email scopes.

If the required claims are not returned in the id token the Relying Party can obtain the additional claims by presenting the access token to the OpenID Provider's userinfo endpoint. This is achieved by sending a HTTP GET request over TLS to the userinfo endpoint URI, passing the access token value in the Authorization header using the Bearer authentication scheme. 

This is illustrated in the example below:

```
  GET /userinfo HTTP/1.1
  Host: server.example.com
  Authorization: Bearer SlAV32hkKG
```

### UserInfo Successful Response

The userinfo claims will be returned in a HTTP 200 OK response as in the example below:

```
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
   "sub": "248289761001",
   "name": "Jane Doe",
   "given_name": "Jane",
   "family_name": "Doe",
   "preferred_username": "j.doe",
   "email": "janedoe@example.com",
   "picture": "http://example.com/janedoe/me.jpg"
  }
```

The sub claim will always be included in the response and this should be verified by the Relying Party to mitigate against token substitution attacks.

For privacy reasons OpenID Providers may elect to not return values for some requested Claims. In that case the claim will be omitted from the JSON object rather than being present with a null or empty string value.

During registration the Relying Party may request that the userinfo response is signed or encrypted in which the claims will be returned in a JWT and the content type will be set to application/jwt. If signed the userinfo Response will contain the iss and aud claims. The Relying Party should validate that the iss value matches OpenID provider's issuer identifier and the aud value contains the Relying Party client id and should

The Relying Party should authenticate the OpenID Provider either by checking the TLS certificate or by validating the signature of the JWT if provided.

### UserInfo Error Response

When an error condition occurs an error response as defined in the [OAuth 2.0 Bearer Token Usage Specification](https://tools.ietf.org/html/rfc6750#section-3.1) will be returned.

An example error response is given below:

```
  HTTP/1.1 401 Unauthorized
  WWW-Authenticate: error="invalid_token",
    error_description="The Access Token expired"
```

## Using Refresh Tokens

An access token is a bearer token that allows any party in possession of it to access a resource without having to prove their identity. Although they are distributed in a safe manner they are typically short lived to protect against them being misused if they are obtained by unauthorized third parties.

A Relying Party may determine that its access token has expired by inspection of the exp claim in the original token response or as a result of receiving an invalid_token error response when attempting to access the userinfo endpoint. 

Should the Relying Party want to refresh the access token it may do so by re-authenticating to the token endpoint and presenting their refresh token. To do this they must a send a HTTP POST request over TLS to the token endpoint as in the example below:

```
  POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded

  client_id=s6BhdRkqt3
  &client_secret=some_secret12345
  &grant_type=refresh_token
  &refresh_token=8xLOxBtZp8
  &scope=openid%20profile
```

The following serialized form parameters must be provided in the request:

|Name|Description|
|----|-----------|
|grant_type|This must be set to refresh_token.|
|refresh_token|The previously received refresh token.|
|scope|The scopes required for the new access token.|

The Relying Party must authenticate to the token endpoint using the authentication method registered for its client_id (see [Client Authentication](#client-authentication)). In the example given above the Relying Party has used the client_secret_post method supplying its client id and secret in the request body.

### Refresh Successful Response

Upon successful validation of the refresh token the OpenID Provider will return a response as for the original token request (see [Token Successful Response](token-successful-response)), with the exception that it might not contain an id token.

An example response is given below:

```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "access_token": "TlBN45jURg",
   "token_type": "Bearer",
   "refresh_token": "9yNOxJtZa5",
   "expires_in": 3600
  }
```

### Refresh Error Response

If the Refresh Request is invalid or unauthorized the OpenID Provider will return a response as for the original request (see [Token Error Response](token-error-response)).

