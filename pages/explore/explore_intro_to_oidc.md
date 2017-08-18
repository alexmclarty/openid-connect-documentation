---
title: Introduction to OpenID Connect
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_intro_to_oidc.html
summary: An introduction to OpenID Connect.
---

## What is OpenID Connect

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

Before attempting to understand the OpenID authentication process it is worth understanding the following key terms:

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
   
