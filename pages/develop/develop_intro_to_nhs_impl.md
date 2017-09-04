---
title: An Overview of the NHS Digital Implementation
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: develop_intro_to_nhs_impl.html
summary: An introduction to the NHS Digital implementation of OpenID Connect.
---

{% include warning.html content="Work in Progress." %}

## Introduction

The previous sections in this site have provided an overview of OpenID Connect and described the Authorization Code Flow in more detail. The purpose of this page is to describe at a high level the current NHS Digital implementation of OpenID Connect.

As an initial step toward offering a fully compliant OpenID Connect solution offering multiple ways to authenticate an End-User, NHS Digital have created an Alpha implementation. This implementation supports a cut down version of OpenID Connect specifically tailored to support native applications deployed on a Windows laptop authenticating users with a Spine smartcard.

This Alpha implementation is described further in the following sections:

## High Level Architecture

The diagram below depicts at a a high level the components involved in the NHS Digital OPenID Connect implementation.

![NHS Digital Implementation](images/OIDCSpineImplementation.jpg)

### Components

The following components are involved in the solution:

* **End-Users Laptop**

  The Alpha implementation is specifically tailored to support third party native applications running on a Windows laptop. To support the solution the laptop must have a smartcard reader and have the Spine Identity Agent installed. The laptop must have N3 connectivity. 
  
* **Identity Agent**

  The Identity Agent is a Windows application that controls the authentication of an End-User by requiring them to enter their smartcard into a reader and enter a passcode, this is done in conjunction with the Spine Authentication Server. Currently this is the only authentication mechanism supported by the NHS Digital.
    
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

![Authorization Code Flow](images/OIDCOnlineFlow.jpg)

As shown above to initiate an authentication the third party application needs to make an OpenId Connect authentication request to the authorization endpoint. This would typically be done by causing the End-User's browser to perform an HTTP GET request. However in this instance both the third party application and Identity Agent Bridge are native applications so the communication is achieved using the Windows Custom URI Scheme. This requires that the Identity Agent Bridge registers a custom URI to receive authentication requests and that the third party application registers a custom URI to receive the response.

When the third party directs an authentication request at the authorize endpoint the Identity Agent Bridge will interact with the Identity Agent to determine whether the End-User has already authenticated. If they haven't this will start the authentication process by displaying a pop-up window requesting the user to enter their smartcard and enter their passcode.

![Spine Login](images/OIDCSpineLogin.jpg)

If the user correctly enters their passcode they will then be requested to select a role from those available to them.

![Spine Role Selection](images/OIDCSpineRoleSelection.jpg)

If the End-User was already authenticated or was newly authenticated the Identity Agent Bridge will interact with the OpenID Connect Server which in turn will interact with the Spine Authentication Server to generate an authorization code. This authorization code will be returned to the third party application by directing an OpenID Connect authentication response to the third party's custom URI. If they authentication fails then an OpenID Connect error response will be returned to the custom URI.

Note that the user is not presented with a screen requesting authorization to release data to the third party. Consent is presumed on the basis of the terms and conditions they signed for use of a smartcard.

The third party native application should return the authorization code to its server component which can then use it to complete a standard Authorization Code Flow by retrieving id, access and refresh tokens from the token endpoint. Note that the current implementation does not support a userinfo endpoint.

### Offline Authentication

{% include warning.html content="Work in Progress." %}

