---
title: The Authorization Code Flow in Detail
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_auth_code_flow.html
summary: A detailed description of the Authorization Code Flow.
---

## Introduction

The Authorization Code Flow is the most commonly used variant of the OpenID Connect authentication flows. It is suited for use with web applications and native applications that utilise a client/server architecture. In this flow rather than return the id, access and refresh tokens directly to the Relying Party's client component an authorization code is returned.

The Relying Party's server component can then exchange the code for the required tokens. This provides the dual benefits of:

1. Not exposing any tokens to the user agent or applications with access to the user agent.
2. Allowing the Relying Party to be authenticated before exchanging the code for tokens.

This flow requires that a Relying Party can securely maintain a client secret between themselves and the OpenID Provider.

## High Level Flow

The diagram below depicts at a a high level the Authorization Code Flow.

![Authorization Code Flow](/images/OIDCAuthCodeFlow.jpg)

1. The Relying Party sends a request to the OpenID Provider to authenticate the End-User. The request must include the Relying Party's identity and the openid scope, it may optionally include other scopes e.g. the email scope if the Relying Party wishes to obtain the user's email address.
2. The OpenID Provider authenticates the End-User using one of the methods available to it and obtains authorization from the End-user to provide the requested scopes to the identified Relying Party.
3. Once the End-User has been authenticated and has authorized the request the OpenID Provider will return an authorization code to the Relying Party's server component.
4. The Relying Party's server component contacts the token endpoint and exchanges the authorization code for an id token identifying the End-User and optionally access and refresh tokens granting access to the userinfo endpoint.
4. Optionally the Relying Party may request the additional user information (e.g. email address) from the UserInfo Endpoint by presenting the access token obtained in the previous step.

This flow is described in much more detail in the following sections.

## Registration

Before being able to interact with an OpenID Provider a Relying Party must register with the provider. During this process the following information, as a minimum, will be exchanged:

1. The URIs of the OpenID Provider's authorization, token and user endpoints.
2. A client identifier uniquely identifying the Relying Party.
3. The Relying Party's redirection URIs to which responses may be redirected.

## Authentication Request

When a Relying Party requires that an End-User is authenticated they should cause a HTTP GET request to be sent from the End-User's user agent to the OpenID Provider's authorization endpoint. This request must be made over TLS.

The request may be generated indirectly via a HTTP 302 redirect response (for example when a user attempts to access a protected resource) or directly e.g. as a result of a login button being hit. An example of each is given below:

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
The OpenID Connect specification defines a number of mandatory and recommended parameters to use in the request. The minimum recommended set is as follows:

|Name|Description|
|----|-----------|
|scope|This is a space delimited list of the scopes requested by the client. It must contain the value openid and may contain other values e.g. profile and email. Unrecognised values will be ignored.|
|response_type|This defines the processing flow to be used when forming the response. When using the Authorization Code Flow, this value must be code.|
|client_id|This must contain the client identifier assigned to the Relying Party during its registration with the OpenID Provider.|
|redirect_uri|This is the URI to which the response should be sent. This must exactly match one of the Relying Party's redirection URIs registered with the OpenID Provider.|
|state|It is recommended that client's use this parameter to maintain state between the request and the callback. Typically, Cross-Site Request Forgery (CSRF, XSRF) mitigation is done by cryptographically binding the value of this parameter with a browser cookie.|

The OpenID Connect specification also defines a number of optional parameters that may be used to modify the behaviour of the authentication process. For example the prompt parameter can be used to control whether the user is prompted for re-authentication or not. 

For more details see the [OpenID Connect Core Specification](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth). It should be noted that support for optional parameters is implementation specific.

## Authentication and Authorization

If the request is valid, the Authorization Server attempts to Authenticate the End-User or determines whether the End-User is Authenticated, depending upon the request parameter values used. The methods used by the Authorization Server to Authenticate the End-User (e.g. username and password, session cookies, etc.) are beyond the scope of this specification. An Authentication user interface MAY be displayed by the Authorization Server, depending upon the request parameter values used and the authentication methods used.

The Authorization Server MUST attempt to Authenticate the End-User in the following cases:

The End-User is not already Authenticated.
The Authentication Request contains the prompt parameter with the value login. In this case, the Authorization Server MUST reauthenticate the End-User even if the End-User is already authenticated.
The Authorization Server MUST NOT interact with the End-User in the following case:

The Authentication Request contains the prompt parameter with the value none. In this case, the Authorization Server MUST return an error if an End-User is not already Authenticated or could not be silently Authenticated.
When interacting with the End-User, the Authorization Server MUST employ appropriate measures against Cross-Site Request Forgery and Clickjacking as, described in Sections 10.12 and 10.13 of OAuth 2.0 [RFC6749].


 TOC 
3.1.2.4.  Authorization Server Obtains End-User Consent/Authorization

Once the End-User is authenticated, the Authorization Server MUST obtain an authorization decision before releasing information to the Relying Party. When permitted by the request parameters used, this MAY be done through an interactive dialogue with the End-User that makes it clear what is being consented to or by establishing consent via conditions for processing the request or other means (for example, via previous administrative consent). Sections 2 and 5.3 describe information release mechanisms.
## Successful Response

## Error Response