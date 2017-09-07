---
title: Offline Authentication in Detail
keywords: explore, design, reference
sidebar: overview_sidebar
permalink: develop_nhs_offline_impl.html
summary: A detailed description of the NHS Digital implementation of OpenID Connect for use whilst offline.
---

## Introduction

This section provides more detailed information on the use of the NHS Digital implementation of OpenID Connect whilst the user's laptop is offline.

## Registration

## Authentication Request

This is as described for the online case with the difference that the third party application must include an initial id_token_hint parameter in the authentication request. This parameter should be set to the value of the id token received as the result of a previous online authentication e.g.

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

1. The id token was issued to the client specified in the request.
2. The id token was issued to the redirect uri specified in the request.
3. The id token was issued for the current device.
4. The id token was issued and signed by the NHS Digital implementation.
5. The user referred to by the id token is still logged in to the laptop with the same seesion i.e. the user has not removed their smartcard since the original authentication took place.

If the validation succeeds then an authorization code will be returned to the third party's redirection uri as for the online case. This is represented in the example below:

```
code=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.ewogICJpc3MiOiJuaHNpZGVudGl0eWFnZW50Oi8vY29ubmVjdC
 9hdXRob3JpemUiLAogICJzdWIiOiJ1aWQ9MjQwMDAwMTA5ODk2LG91PVBlb3BsZSxvPW5ocyIsCiAgImlhdCI6M
 TUwNDc4MzYzOCwKICAiZXhwIjoxNTA0NzgzNjY4LAogICJhdWQiOiJhYmMxMjMiLAogICJqdGkiOiJlYzUyZjIz
 YzIyMmE0ZWY3YjQ5OGM3ZTAzYTRhYTdmOCIKfQ. ...
state=af0ifjsldkj
```

#### Authorization Code Validation

In the offline scenario the third party application is not able to pass the authorization code to it's server to perform a get token request so it has to rely on a successful response to confirm that the user is still authenticated.

In a break with the OpenID Connect specification it may also perform further validation by inspection of the authorization code which currently uses a JWT format.

##### Header

Decoding the first element of the JWT reveals the following header:

```
{"typ":"JWT","alg":"HS256"}
```

#### JSON Object

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

### Authentication Error Response

