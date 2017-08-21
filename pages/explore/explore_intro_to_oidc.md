---
title: An Introduction to OpenID Connect
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_intro_to_oidc.html
summary: An introduction to the main concepts of OpenID Connect.
---

## Introduction

OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol. It allows Clients to verify the identity of an End-User based on the authentication performed by an Authorization Server, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

OpenID Connect allows clients of all types, including Web-based, mobile, and JavaScript clients, to request and receive information about authenticated sessions and end-users. The specification suite is extensible, allowing participants to use optional features such as encryption of identity data, discovery of OpenID Providers and session management.

The full set of specifications describing OpenID Connect can be found at <http://openid.net/connect/>.

### Actors

The following actors take part in an OpenID Connect authentication:

* **OpenID Provider (OP)**

  An OAuth 2.0 Authorization Server that is capable of authenticating an End-User and providing information to a Relying Party about the authentication event and the End-User.

* **Relying Party (RP)**

  The client application requesting End-User authentication and information about the End-User.
  
* **End-User**

  The human participant that being authenticated and about whom the Relying Party is requesting information.

### Key Terms

Before attempting to understand the OpenID Connect authentication process it is worth understanding the following key terms:

* **Claim**

  A piece of information asserted about an entity. The OpenID Connect defines a number of standard claims e.g. the name claim represents an End-User's full name in displayable format.
  
* **Scope**

  A collection of claims. The OpenID Connect profile defines a number of standard scopes that a Relying Party can request about an authentication event or End-User. For example the profile scope contains: name, family_name, given_name etc.

* **ID Token**

  A JSON Web Token ([JWT](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32)) that contains claims about the authentication event and may contain claims about the End-User. An ID Token is requested using the openid scope.

* **Access Token**

  A credential used to access protected resources. Access tokens represent specific scopes and durations of access.
   
* **Refresh Token**

  A credential used to obtain access tokens.  Refresh tokens are issued to the client by an authorization server and are used to obtain a new access token when the current access token becomes invalid or expires.
   
* **UserInfo Endpoint**

  A protected Resource that, when presented with an access token by the RP returns authorized information about an End-User.
Access Token Scope
   
## High Level Process Flow

The diagram below depicts at a a high level the OpenID Connect authentication process.

![OpenID Connect High Level Flow](/images/OIDCHighLevelFlow.jpg)

1. The Relying Party sends a request to the OpenID Provider to authenticate the End-User. The request must include the Relying Party's identity and the openid scope, it may optionally include other scopes e.g. the email scope if the Relying Party wishes to obtain the user's email address.
2. The OpenID Provider authenticates the End-User using one of the methods available to it and obtains authorization from the End-user to provide the requested scopes to the identified Relying Party.
3. Once the End-User has been authenticated and has authorized the request the OpenID Provider will respond to the Relying Parting with an id token identifying the End-User and optionally with access and refresh Tokens granting access to the UserInfo Endpoint.
4. Optionally the Relying Party may request the additional user information (e.g. email address) from the UserInfo Endpoint by presenting the access token obtained in the previous step.

The above gives a generic flow however there are three variations of this flow as described below.

### Authentication Flow Types

OpenID Connect defines three types of authentication flow to cater for different client types: the Authorization Code Flow, the Implicit Flow and the Hybrid Flow. These are briefly described below with the Authorization Code Flow being described in more detail in a later [section](#authorization-code-flow).

#### Authorization Code Flow

This is the most commonly used flow and is used with web applications as well as native applications that utilise a client/server architecture. In this flow rather than return the id, access and refresh Tokens directly to the Relying Party's client component an authorization code is returned. The Relying Party's server component can then exchange the code for the required tokens. This provides the dual benefits of:

1. Not exposing any tokens to the user agent or applications with access to the user agent.
2. Allowing the Relying Party to be authenticated before exchanging the code for tokens.

This flow requires that a Relying Party can securely maintain a client secret between themselves and the OpenID Provider.

#### Implicit Flow

The Implicit Flow is mainly used by Relying Parties implemented as a client in a browser using a scripting language or as a native application that does not utilise a client/server architecture. In this flow tokens are returned directly to the client. This is a less secure mechanism as this may may expose the tokens to the End-User or applications that have access to the user agent. In this flow the OpenID Provider does not authenticate the Relying Party.

#### Hybrid Flow

This is a rarely used flow that allows the Relying Party's client and server components to receive tokens separately from one another. Essentially it is a combination of the code and implicit flows.

## Data Artefacts

This section describes in more detail some of the data artefacts used in the flow above.

### ID Token

The id token is a JSON Web Token [JWT](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32)) that contains claims about the authentication of an End-User and optionally data about the End-User.

As a minimum the id token contains the following claims:

Claim|Name|Description|
-----|----|-----------|
iss|Issuer Identifier|An identifier for the issuer of the response.|
sub|Subject Identifier|A unique identifier for the End-User.|
aud|Audience(s)|The identifier of the Relying Party and any other parties intended as a recipient.|
exp|Expiration|The time on or after which the ID Token must not be accepted for processing.|
iat|Issuance Time|The time at which the JWT was issued.|

The id token may additionally contain other claims:

1. There are a number of optional standard claims that may be returned depending on the flow and the parameter provided in the initial request e.g. a relying Party may supply a nonce parameter in the original request and this will be returned unmodified in a nonce claim to allow the Relying Party to mitigate against a replay attack.

2. A OpenID Provider implementation may optionally decide to return additional user information in the ID Token e.g. a name claim containg the End-User's full name.

For more details see the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#IDToken).

The claims are represented in a simple JSON object e.g.

```
{
 "iss": "https://server.example.com",
 "sub": "24400320",
 "aud": "s6BhdRkqt3",
 "nonce": "n-0S6_WzA2Mj",
 "exp": 1311281970,
 "iat": 1311280970,
 "auth_time": 1311280969,
 "acr": "urn:mace:incommon:iap:silver"
}
```
The JSON object is signed using a JSON Web Signature [JWS](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41) and optionally they encrypted using JSON Web Encryption [JWE](http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption). Signing the token allows the integrity and origin of the token to be validated by the Relying Party whilst encrypting the token provides confidentiality.

Finally the id token header, JSON claims and signature are encoded into a base 64 URL-safe string e.g.

```
"eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
```

### Access and Refresh Tokens

Access tokens are credentials used to access protected resources. An access token represents specific scopes and durations of access, granted by the resource owner, and enforced by the resource server and authorization. OAuth 2.0 supports a number of access token types, the type used in OpenID Connect are bearer tokens which can be simply understood as meaning "give access to the bearer of this token". 

Access tokens can have different formats, structures, and methods of utilization based on the resource server security requirements. However they are represented as a string the structure of which is usually opaque to the client. An example of a returned access token is given below:

```
access_token=jHkWEdUXMU1BwAsC4vtUsZwnNvTIxEl0z9K3vx5KF0Y
```

Refresh tokens are credentials used to obtain access tokens.  Refresh tokens are issued to the client by the authorization server and are  used to obtain a new access token when the current one becomes invalid or expires, or to obtain additional access tokens with identical or narrower scope. Unlike access tokens, refresh tokens are intended for use only with authorization servers and are never sent to resource servers. As for access tokens a refresh token is represented as a string that is usually opaque to the client e.g. 

```
refresh_token=9yNOxJtZa5
```  
   
### UserInfo

As described above the ID Token principally holds claims about the authentication event and the identity of the End-User. A Relying Party  wishing to obtain further data about the End-User can do this by presenting the access token they obtained to the UserInfo Endpoint. A successful request will result in a JSON object containing claims about the End-User being returned. As a minimum this will always contain the sub claim which the Relying Party must verify to protect against a token substitution attack. An example JSON object is given below:

```
  {
   "sub": "248289761001",
   "name": "Jane Doe",
   "given_name": "Jane",
   "family_name": "Doe",
   "preferred_username": "j.doe",
   "email": "janedoe@example.com",
  }
```
