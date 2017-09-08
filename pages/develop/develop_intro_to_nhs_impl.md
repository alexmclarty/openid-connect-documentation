---
title: An Overview of the NHS Digital Implementation
keywords: explore, design, reference
sidebar: overview_sidebar
permalink: develop_intro_to_nhs_impl.html
summary: An introduction to the NHS Digital implementation of OpenID Connect.
---

## Introduction

The previous sections in this site have provided an overview of OpenID Connect and described the Authorization Code Flow in more detail. The purpose of this page is to describe at a high level the current NHS Digital implementation of OpenID Connect.

As an initial step toward offering a fully compliant OpenID Connect solution offering multiple ways to authenticate an End-User, NHS Digital have created an Alpha implementation. This implementation supports a cut down version of OpenID Connect specifically tailored to support native applications deployed on a Windows laptop authenticating users with a Spine smartcard.

This Alpha implementation is described further in the following sections:

## High Level Architecture

The diagram below depicts at a a high level the components involved in the NHS Digital OpenID Connect implementation.

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

  The OpenID Connect Server is a new component that allows a third party to complete an OpenID Connect authorization code flow. It presents token, JWKS and configuration endpoints.

* **Third Party Application**

  OpenID Connect may be most commonly used by web applications but the current solution is specifically tailored to support third party applications using a client/server architecture comprising a native windows application implementing presentation logic and a server component implementing business logic.
  
  Support for this particular architecture has been provided to allow the development of mobile applications where the user can continue to work even when the laptop is no longer online. Further information on the offline capabilities of this solution is given in later sections.
  
## High Level Flows

This solution supports both online and offline authentication as described below:

### Online Authentication

Online authentication is provided using a tailored Authorization Code Flow as depicted below:

![Online Authentication Flow](images/OIDCOnlineFlow.jpg)

1. The Relying Party sends a request to the OpenID Provider's authorization endpoint to authenticate the End-User. The request must include the Relying Party’s identity and the openid scope. 

   Normally this would be done by causing the End-User's browser to perform an HTTP GET request. However in this instance both the Relying Party client and authorization endpoint are native applications so the communication is achieved using the Windows Custom URI Scheme.
2. The OpenID Provider authenticates the user by requiring them to insert their smartcard and enter their passcode.
3. Once the End-User has been authenticated the OpenID Provider will return an authorization code to the Relying Party’s server component. 
   
   Normally this would be done via a HTTP 302 redirect request. However in this instance both the Relying Party client and authorization endpoint are native applications so the response is returned using the Windows Custom URI Scheme. The Relying Party client will then communicate the authorization code to its server component.
4. The Relying Party’s server component contacts the token endpoint and exchanges the authorization code for an id token identifying the End-User.

#### Authentication and Authorization

When the third party directs an authentication request at the OpenID Provider's authorize endpoint the Identity Agent Bridge will interact with the Identity Agent to determine whether the End-User has already authenticated. If they haven't this will start the authentication process by displaying a pop-up window requesting the user to enter their smartcard and enter their passcode.

![Spine Login](images/OIDCSpineLogin.jpg)

If the user correctly enters their passcode they will then be requested to select a role from those available to them.

![Spine Role Selection](images/OIDCSpineRoleSelection.jpg)

If authentication was successful or the user was already authenticated the Identity Agent Bridge will interact with the OpenID Connect Server (which in turn will interact with the Spine Authentication Server) to generate an authorization code. This authorization code will be returned to the third party application by directing an OpenID Connect authentication response to the third party's custom URI. If the authentication fails then an OpenID Connect error response will be returned to the custom URI.

{% include important.html content="Note that unlike the standard flow authorization code flow described [here](explore_auth_code_flow#authentication-and-authorization) the user is not presented with a screen requesting authorization to release the requested data to the third party. Consent is presumed on the basis of the terms and conditions signed for use of a smartcard." %}

The third party native application should return the authorization code to its server component which can then use it to complete a standard Authorization Code Flow by retrieving id, access and refresh tokens from the token endpoint. Note that the current implementation does not support a userinfo endpoint.

### Offline Authentication

The NHS Digital implementation of OpenID Connect has been specifically tailored to support the use case of a user working offline if they have first successfully authenticated whilst online. One such scenario where this use case might implemented is given below:

1. At the beginning of the day user accesses N3 and starts up their community healthcare worker application.
2. The application authenticates the user using the NHS Digital OpenID Connect online implementation (this requires the user to insert their smartcard and enter their passcode) and downloads the workers appointments for the day including the details of the patients they are to visit.
3. The worker commences their visits during which time they will no or limited N3 access. During a visit the worker requests the application to view a patient's details.
4. The application validates that the user is still authenticated using the NHS Digital OpenID Connect offline implementation (detailed further below) and allows the user to view the patient details and enter notes on the visit.
5. At the end of the day the user again accesses N3 and requests the application to upload the notes taken during the day.
6. The application re-authenticates the user using the NHS Digital OpenID Connect online implementation and uploads the notes.

{% include warning.html content="The implementation details for the offline use case as described below may change in future versions." %}

The offline authentication step described in step 4 is provided by a truncated Authorization Code Flow as depicted below:

![Offline Authentication Flow](images/OIDCOfflineFlow.jpg)

1. The Relying Party sends a request to the OpenID Provider's authorization endpoint to authenticate the End-User. The request must include the Relying Party’s identity, the openid scope and the id token returned when the user first authenticated. 

   Normally this would be done by causing the End-User's browser to perform an HTTP GET request. However in this instance both the Relying Party client and authorization endpoint are native applications so the communication is achieved using the Windows Custom URI Scheme.
2. The OpenID Provider validates that the user is still authenticated with the session as for the provided id token.
3. Once the End-User has been authenticated the OpenID Provider will return an authorization code to the Relying Party. 
   
   Normally this would be done via a HTTP 302 redirect request. However in this instance both the Relying Party client and authorization endpoint are native applications so the response is returned using the Windows Custom URI Scheme. The Relying Party client can isnspect the authorization code to confirm that the user is still authenticated.

#### Authentication and Authorization

To support the offline use case the NHS Digital Identity Agent v2 is required. This provides two new modes of operation:

* **Session Lock Persistence** 

  If a user removes their Smartcard in order to temporarily leave their workstation, they are able to ‘lock’ their Spine session. On re-insertion of their Smartcard, the user is able to re-authenticate and continue their Spine session, with no loss of state.

* **Mobility Mode** 

  This mode enables users of mobile devices running a Windows OS to authenticate, remove their Smartcard from the device for secure storage (lanyard etc.), and continue working as normal.

The behaviour of the Identity Agent when the third party attempts to validate whether the user is still validated will depend on which modes have been enabled and how they have configured. The validation maky occur seamlessly or may result in the Login screen illustrated above being re-displayed requiring the user to enter their passcode.

For full details of the two modes and how they may be configured see the supporting documentation for the Identity Agent which may be found [here](http://nww.hscic.gov.uk/dir/downloads/).
