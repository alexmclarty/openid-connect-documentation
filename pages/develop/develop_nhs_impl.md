---
title: NHS Digital Implementation in Detail
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: develop_nhs_impl.html
summary: A detailed description of the NHS Digital implementation of OpenID Connect.
---

{% include warning.html content="Work in Progress." %}

## Introduction

This section provides more detailed information on the use of the NHS Digital implementation of OpenID Connect.

## Registration

Before using the NHS Digital implementation of OpenID Connect a third party will need to register their details. This is a manual step as [Dynamic Registration](explore_other_features#dynamic_registration) is not supported. During this step the following values will be agreed:

|Value|Description|
|-----|-----------|
|client_id|An identifier for the third party application which will be allocated by NHS Digital e.g. abc123.|
|client_name|A name for the third party application.|
|redirect_uris|The URI(s) to which the third party application may request authorization codes to be redirected. As noted above these will be custom URIs e.g. clientapp://connect/authresponse|
|jwks_uri|The URI which the third party will us as a JWKS endpoint ([see key management](explore_other_features#key_management)) e.g. https://client.example.org/.well-known/jwks.json.|

## Discovery

The NHS Digital implementation does not support [Discovery](explore_other_features#discovery) but it does expose a URI where the configuration data for the OpenID Provider can be retrieved e.g.

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

## Key Management

### NHS Digital Keys

The NHS Digital solution provids a JWKS endpoint ([see key management](explore_other_features#key_management)). The value taken from the JSON document above is:

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

### Third Party Keys

When submitting a token request to the NHS Digtal token endpoint third party applications are required to authenticate themselves ([see client authentication](explore_auth_code_flow#client-authentication)). 

The NHS Digital implementation requires that this is done using the private_key_jwt method using the RSA256 algorithm. To support this mechanism the third party must generate its own RSA key pairs and publish the public key via a JWKS endpoint in the same fashion as described for the NHS Digital public keys above.

The URI for the third party JWKS will be recorded as part of the registration process. The URI must be accessible to NHS Digital via N3.

The third party should periodically rotate their signing keys using the three step process as described above for the NHS Digital public keys. 

## Authentication Request

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#authentication_request) with the exception that the request should be made via use of the Windows Custom URI Scheme to the Identity Agent Bridge custom URI e.g.

```
nhsidentityagent://connect/authorize?
   scope=openid&
   response_type=code&
   client_id=abc123&
   redirect_uri=clientapp%3A%2F%2Fconnect%2authresponse&
   state=af0ifjsldkj
```

The current Identity Agent Bridge implementation currently only supports the scope, response_type, client_id, redirct_uri and state parameters. The only scope supported is openid.

### Authentication Successful Response

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#authentication-successful-response) with the exception that the response will be returned to the specified redirect_uri via the Windows Custom URI Scheme.

An example response from the NHS Digital implementation is given below:

```
code=eyJraWQiOiJyRi1Ebzh4aGFqTC1iZnJDMEVmMERwcXZ3Yko5VVV6VWJFRlF3bDQycnc0IiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYif
     Q.eyJpdiI6IlRXMTljcEdYbGlLWDJXV01ObElHZjc1ZE5rTzBpbFBTOGthOUNkLUVublE9Iiwic3ViIjoidWlkPTI0MDAwMDEwOTg5Nix
     vdT1QZW9wbGUsbz1uaHMiLCJzc29fdGlja2V0IjoiY2FIbjNLaTRBdVVySEQ4TWdkUFBMbk1iRUtDOGRfeHZEVnZXX3FjR2hvZHJqR29z
     bFRSaThDVjdEb1FsS3dYY1hzeUpUekFaWG02RkN0STE5OEk9IiwiaXNzIjoiaHR0cHM6Ly9jb25uZWN0LmlhbS5zcGluZTIubmNycy5ua
     HMudWsvYXV0aG9yaXplIiwianRpIjoiYmRhYjk1NzI0NDUwNDEyZjhiMTUwMzNhNzg1ZjI3YmEiLCJhenAiOiJzNkJoZFJrcXQzIiwiaW
     F0IjoxNTA0NjAyNTc4LCJhbXIiOlsicGluIiwibGVnYWN5X3NpZ25lZF9jaGFsbGVuZ2UiXSwiYXVkIjoiaHR0cHM6Ly9jb25uZWN0Lml
     hbS5zcGluZTIubmNycy5uaHMudWsvdG9rZW4iLCJyb2xlX2lkIjoiMjQwMDAwMTE1ODk0IiwiZGV2aWNlX2lkIjoiQU11dmM3SDlUbHJp
     Q24xUndqejVqV1VBVnZBcWh5bUoiLCJleHAiOjE1MDQ2MDI2MDgsInRhZyI6IjR1SUtrSDh3S1diUkwwb0F3REZyb0E9PSJ9.vo3Gqz-g
     QIKYvqIzBaiJepTVPfD74vlkxgQSvq3KmHEMx5IMUn0qZ7MOFwbfv8S1AVX1tTCSbOUpKSD2Y1Zxdo89Lr5WCI4dSAR8ACazZILY89Ra5
     xQXr6yGjPXfCyg_Nqt9HYt1aFWCFt4QmeLNBxQgv1wCLEVFdKIagEl3THNK_Nlq9PESngi_RId8xZjXedpjjXsf2Vf4BtweJEbFFIoW0X
     S1h91tGTQ0YTIBwgFg_VAQh-0NNeDDKtwA7dcGxV2orqq9pv6MoCa_9i2R_QYgs19I02Y15Ggd3_TifCqi2FKtshbZJFB5Zm6vYg4a21p
     kuus_s5ZTsFxbo7b0aA&
state=af0ifjsldkj
```

The NHS Digital implementation currently uses a JWT token with encoded state information as its authorization code. However as per the OpenID Connect specification and to protect against any future changes to its structure third party applications should treat the code as an opaque value.

### Authentication Error Response

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#authentication-error-response) with the exception that the response will be returned to the specified redirect_uri via the Windows Custom URI Scheme.

The NHS Digital implementation only supports the error and state parameters, an error_description parameter will not be provided.

An example response from the NHS Digital implementation is given below:

```
error=invalid_request
&state=af0ifjsldkj
```

{% include warning.html content="Need to check the errors returned." %}

## Token Request

Once the third party native application has received an authorization code it can pass it to it's server component to request the NHS Digital OpenID Connect Server to provide the associated id, access and refresh tokens. 

This is done by sending a token request to the NHS Digital token endpoint as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#token-request).

The token endpoint can be obtained from the configuration data. From the example data above this would be:

```
https://connect.iam.spine2.ncrs.nhs.uk/token
``` 

An example of a request is given below. The example shows the authorization code being supplied along with the client_id and redirect_uri from the original request, the example the client assertion JWT value has been truncated:

```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded

  grant_type=authorization_code&
  code=eyJraWQiOiJyRi1Ebzh4aGFqTC1iZnJDMEVmMERwcXZ3Yko5VVV6VWJFRlF3bDQycnc0IiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYif
       Q.eyJpdiI6IlRXMTljcEdYbGlLWDJXV01ObElHZjc1ZE5rTzBpbFBTOGthOUNkLUVublE9Iiwic3ViIjoidWlkPTI0MDAwMDEwOTg5Nix
       vdT1QZW9wbGUsbz1uaHMiLCJzc29fdGlja2V0IjoiY2FIbjNLaTRBdVVySEQ4TWdkUFBMbk1iRUtDOGRfeHZEVnZXX3FjR2hvZHJqR29z
       bFRSaThDVjdEb1FsS3dYY1hzeUpUekFaWG02RkN0STE5OEk9IiwiaXNzIjoiaHR0cHM6Ly9jb25uZWN0LmlhbS5zcGluZTIubmNycy5ua
       HMudWsvYXV0aG9yaXplIiwianRpIjoiYmRhYjk1NzI0NDUwNDEyZjhiMTUwMzNhNzg1ZjI3YmEiLCJhenAiOiJzNkJoZFJrcXQzIiwiaW
       F0IjoxNTA0NjAyNTc4LCJhbXIiOlsicGluIiwibGVnYWN5X3NpZ25lZF9jaGFsbGVuZ2UiXSwiYXVkIjoiaHR0cHM6Ly9jb25uZWN0Lml
       hbS5zcGluZTIubmNycy5uaHMudWsvdG9rZW4iLCJyb2xlX2lkIjoiMjQwMDAwMTE1ODk0IiwiZGV2aWNlX2lkIjoiQU11dmM3SDlUbHJp
       Q24xUndqejVqV1VBVnZBcWh5bUoiLCJleHAiOjE1MDQ2MDI2MDgsInRhZyI6IjR1SUtrSDh3S1diUkwwb0F3REZyb0E9PSJ9.vo3Gqz-g
       QIKYvqIzBaiJepTVPfD74vlkxgQSvq3KmHEMx5IMUn0qZ7MOFwbfv8S1AVX1tTCSbOUpKSD2Y1Zxdo89Lr5WCI4dSAR8ACazZILY89Ra5
       xQXr6yGjPXfCyg_Nqt9HYt1aFWCFt4QmeLNBxQgv1wCLEVFdKIagEl3THNK_Nlq9PESngi_RId8xZjXedpjjXsf2Vf4BtweJEbFFIoW0X
       S1h91tGTQ0YTIBwgFg_VAQh-0NNeDDKtwA7dcGxV2orqq9pv6MoCa_9i2R_QYgs19I02Y15Ggd3_TifCqi2FKtshbZJFB5Zm6vYg4a21p
       kuus_s5ZTsFxbo7b0aA&
  redirect_uri=clientapp%3A%2F%2Fconnect%2authresponse&
  client_id=abc123&
  client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
  client_assertion=eyJhbGciOiJSUzI1NiIsImtpZCI6IjEwMCJ9.a ... b.c ... d
```

### Client Authentication

The NHS Digital implementation requires that the client authenticates using the private_key_jwt method using the RSA256 algorithm. 

To support this mechanism the third party must generate its own RSA key pairs and publish the public key via a JWKS endpoint in the same fashion as described for the NHS Digital public keys above.

```
{"alg":"RS256","kid":"100"}
```

```
{
  "iss": "abc123",
  "sub": "abc123",
  "aud": "https://connect.iam.spine2.ncrs.nhs.uk/token",
  "iat": 1504611135668,
  "exp": 1504611165668,
  "jti": "jasdh87je9p8rsdj23392389823"
}
```

#### ID Token

```
{"role":"S0080:G0450:R5080","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","jti":"0cca2fe5675f4fe28f687ec5a2dac187","activities":["B0005","B0008"],"exp":1503584507}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - Refresh Token Decoded: {"iv":"LopflBFmvq3BJRUm5r2v3cGpVE7iEyLN4Qf8YLvXS4o=","role_id":"240000115894","device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","sso_ticket":"3zkYKhNMgbFwVqAy0aBS8W93epK8k4xsBWP-8sbns0SJmr1z2B5hFJHbe8txQQ2eluxmULLB2ogxHYpZvyQ=","jti":"2617f03c0b44454b93b427c5d77e704d","tag":"dkQryHQjQ1KrYlFDaNz_3Q==","exp":1503620477}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - ID Token Decoded: {"device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","role":{"role_id":"240000115894","name":"\"Admin & Clerical\":\"Management - A & C\":\"Registration Authority Manager\"","code":"S0080:G0450:R5080"},"activities":["B0005","B0008"],"amr":["pin","legacy_signed_challenge"],"jti":"2b44cf3602d445e0909a808ad3194341","name":"Seven User Mr","s_hash":"LLJ6GrrF3GenakaKAyYBTg","at_hash":"gNn8ijUTnwM34MxSFzBgYw","sub":"uid=240000109896,ou=People,o=nhs","org":{"name":"GREATER MANCHESTER STRATEGIC HA","code":"Q14"},"iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"s6BhdRkqt3","exp":1503620477,"c_hash":"MPZJ1jWGHrGqL-fMHAEBMQ"}
```

