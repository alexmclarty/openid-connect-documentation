---
title: Offline Authentication in Detail
keywords: explore, design, reference
sidebar: overview_sidebar
permalink: develop_nhs_offline_impl.html
summary: A detailed description of the NHS Digital implementation of OpenID Connect for use whilst offline.
---

## Introduction

This section provides more detailed information on the use of the NHS Digital implementation of OpenID Connect whilst the user's laptop is offline.

{% include warning.html content="The implementation details for the offline use case as described below may change in future versions." %}

## Registration

For third party applications wishing to implement the offline use case an additional value will be agreed during the registration process as follows:

|Value|Description|
|-----|-----------|
|client_secret|A shared secret known to both NHS Digital and the third party application e.g. ZJYCqe3Ge57GRvdrAxudKyZS0XhGvjk7.|


## Discovery

This is as described for the online use case.

## Key Management.

This is as described for the online use case.

## Authentication Request

This is as described for the online use case with the difference that the third party application must include an initial id_token_hint parameter in the authentication request. This parameter should be set to the value of the id token received as the result of the previous online authentication e.g.

```
nhsidentityagent://connect/authorize?
   scope=openid&
   response_type=code&
   client_id=abc123&
   redirect_uri=clientapp%3A%2F%2Fconnect%2authresponse&
   state=af0ifjsldkj
   id_token_hint=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InJGLURvOHhoYWpMLWJmckMwRWYwRHBxdndiSjlVVXpVYkVGUXds
                 NDJydzQifQ.eyJhdF9oYXNoIjoiRTRGYXlDMHVxd21DaFJpcUVyb01SZyIsImlhdCI6MTUwNDc4MjE2MywiYWN0aXZpdGllc
                 yI6WyJCMDAwNSIsIkIwMDA4Il0sImp0aSI6ImE2N2NmZjE4ZGFlZTQ2MzJiN2Y1NWIxMDJiMDFhMGM1IiwiY19oYXNoIjoiS
                 UhmOFUxM1kwSWctNEdaaEJHVlBMdyIsImFtciI6WyJwaW4iLCJsZWdhY3lfc2lnbmVkX2NoYWxsZW5nZSJdLCJvcmciOnsib
                 mFtZSI6IkdSRUFURVIgTUFOQ0hFU1RFUiBTVFJBVEVHSUMgSEEiLCJjb2RlIjoiUTE0In0sImRldmljZV9pZCI6IkFNdXZjN
                 0g5VGxyaUNuMVJ3ano1aldVQVZ2QXFoeW1KIiwic3ViIjoidWlkPTI0MDAwMDEwOTg5NixvdT1QZW9wbGUsbz1uaHMiLCJyb
                 2xlIjp7InJvbGVfaWQiOiIyNDAwMDAxMTU4OTQiLCJuYW1lIjoiXCJBZG1pbiAmIENsZXJpY2FsXCI6XCJNYW5hZ2VtZW50I
                 C0gQSAmIENcIjpcIlJlZ2lzdHJhdGlvbiBBdXRob3JpdHkgTWFuYWdlclwiIiwiY29kZSI6IlMwMDgwOkcwNDUwOlI1MDgwI
                 n0sInNfaGFzaCI6ImpiLVgtR0ZmbDRjTmg4Zjd5VEdjdEEiLCJhdWQiOiJzNkJoZFJrcXQzIiwiaXNzIjoiaHR0cHM6Ly9jb
                 25uZWN0LmlhbS5zcGluZTIubmNycy5uaHMudWsvdG9rZW4iLCJleHAiOjE1MDQ4MTgxNjMsIm5hbWUiOiJTZXZlbiBVc2VyI
                 E1yIn0.ce9ezerv2IIW6q7gbDTf6ZTYVOLhsbdoXwvRn-kbY_PYa0THqMb21QFtWbTi7WpOgs4iun4lW_zfXYDvib-bcfr4m
                 YfD2hdgjl69g12DNPfPgB-gdHqFxo5KPqMNm74hOTke5kMXWPS9ozeJIcwA1XVQk_ZiEH_7Qxd-YHXr_TDewLEkOPp4zRoHb
                 lrPplPL4qKl_L1ATACP86nIRPtBO2KgOX0SDu9mP7QExD5chqktb3NgZunT9pAtCcyFF01usNSVgkWiUMax5FOzbBVM5TING
                 Y3k3pIflD6IaU4vE66muFKAKSDUUqGbhSBgp8NsWLuq7m382i4QtmSDU6LO4A
```


### Authentication Successful Response

The NHS Digital implementation will check the parameters passed in the normal way and will additionally validate that:

1. The laptop is offline. 
2. The id token was issued to the client specified in the request.
3. The id token was issued to the redirect uri specified in the request.
4. The id token was issued for the current device.
5. The id token was issued and signed by the NHS Digital implementation.
6. The user referred to by the id token is still logged in to the laptop with the same seesion i.e. the user has not removed their smartcard since the original authentication took place.

If the validation succeeds then an authorization code will be returned to the third party's redirection uri as for the online use case. An example response is given below (the code value has been truncated for brevity):

```
code=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.ewogICJpc3MiOiJuaHNpZGVudGl0eWFnZW50Oi8vY29ubmVjdC
 9hdXRob3JpemUiLAogICJzdWIiOiJ1aWQ9MjQwMDAwMTA5ODk2LG91PVBlb3BsZSxvPW5ocyIsCiAgImlhdCI6M
 TUwNDc4MzYzOCwKICAiZXhwIjoxNTA0NzgzNjY4LAogICJhdWQiOiJhYmMxMjMiLAogICJqdGkiOiJlYzUyZjIz
 YzIyMmE0ZWY3YjQ5OGM3ZTAzYTRhYTdmOCIKfQ. ...
state=af0ifjsldkj
```

### Authorization Code Validation

Whilst offline it is not possible to complete an authorization code flow by performing a get token request. As a result the third party application must rely on a successful response to confirm that the user is still authenticated.

However in contravention of the OpenID Connect specification the third party application may perform further validation of the response by inspection of the returned authorization code. This code is returned as a signed JWT comprising a Base64URL encoded header, JSON object and signature separated by . characters. This is described further below.

#### Header

Decoding the first element of the JWT reveals the following header:

```
{"typ":"JWT","alg":"HS256"}
```

This specifies that the NHS Digital implementation has signed the token with a HMAC Signature utilising the SHA-256 hashing algorithm. This will have been done using the shared secret agreed during registration of the third party application.

#### JSON Object

The second element gives the JSON object containing the claims about the user. Decoding the the value from the example above we get the string below:
```
{
  "iss":"nhsidentityagent://connect/authorize",
  "sub":"uid=240000109896,ou=People,o=nhs",
  "iat":1504783638,
  "exp":1504783668,
  "aud":"abc123",
  "jti":"ec52f23c222a4ef7b498c7e03a4aa7f8"
}
```

The NHS Digital implementation will return the following claims in the JSON:

|Name|Description|
|----|-----------|
|iss|The identity of the NHS Digital implementation e.g. nhsidentityagent://connect/authorize.|
|sub|The authenticated user's identity. Set to the user's Distinguished Name (DN) as retrieved from the directory e.g. uid=240000109896,ou=People,o=nhs.|
|iat|The time at which the JWT was issued represented as the number of seconds from 1970-01-01T0:0:0Z as measured in UTC.|
|exp|The time at which the token was issued plus 30 seconds.|
|jti|A unique identifier for the token.|

#### Signature

The third party application may confirm the integrity of the authorization code JWT by validating the signature element. As specified by the header this will have been created using the HS256 algorithm and the shared secret agreed during the registration process.

Details on how the signature will have been created and how to validate it can be found in the [JSON Web Token Specification](https://tools.ietf.org/html/rfc7519) and the [JSON Web Algorithms](https://tools.ietf.org/html/rfc7518).

### Authentication Error Response

This is as described for the online use case.

For the offline use case the NHS Digital implementation supports the following errors.

|Error|
|-----|
|invalid_request|

## Token Request

As described above the token request is not supported in offline mode.

## UserInfo Request

A userinfo endpoint is not supported by the NHS Digital implementation.

## Using Refresh Tokens

The use of refresh tokens is not supported by the NHS Digital implementation.



