---
title: Online Authentication in Detail
keywords: explore, design, reference
sidebar: overview_sidebar
permalink: develop_nhs_impl.html
summary: A detailed description of the NHS Digital implementation of OpenID Connect for use whilst online.
---

## Introduction

This section provides more detailed information on the use of the NHS Digital implementation of OpenID Connect whilst the user's laptop is connected to N3.

## Registration

Before using the NHS Digital implementation of OpenID Connect a third party will need to register their details. This is a manual step as [Dynamic Registration](explore_other_features#dynamic_registration) is not supported. During this step the following values will be agreed:

|Value|Description|
|-----|-----------|
|client_id|An identifier for the third party application which will be allocated by NHS Digital e.g. abc123.|
|client_name|A name for the third party application.|
|redirect_uris|The URI(s) to which the third party application may request authorization codes to be redirected. As noted above these will be custom URIs e.g. clientapp://connect/authresponse|
|jwks_uri|The URI which the third party will us as a JWKS endpoint ([see Key Management](explore_other_features#key_management)) e.g. https://client.example.org/.well-known/jwks.json.|

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

The NHS Digital solution provids a JWKS endpoint ([see Key Management](explore_other_features#key_management)). The value taken from the JSON document above is:

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

When submitting a token request to the NHS Digtal token endpoint third party applications are required to authenticate themselves ([see Client Authentication](explore_auth_code_flow#client-authentication)). 

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

Submitting an authentication request will initiate the user interaction described in the [High Level Flows](develop_intro_to_nhs_impl#high_level_flows) section of the introductory page.

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

The NHS Digital implementation currently uses a JWT token with encoded state information as its authorization code. However to conform to the OpenID Connect specification and to protect against any future changes to its structure third party applications should treat the code as an opaque value.

### Authentication Error Response

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#authentication-error-response) with the exception that the response will be returned to the specified redirect_uri via the Windows Custom URI Scheme.

The NHS Digital implementation only supports the error and state parameters, an error_description parameter will not be provided.

An example response from the NHS Digital implementation is given below:

```
error=invalid_request
&state=af0ifjsldkj
```
The NHS Digital implementation supports the following errors.

|Error|
|-----|
|invalid_request|
|access_denied|
|internal_server_error|

## Token Request

Once the third party native application has received an authorization code it can pass it to it's server component to request the NHS Digital OpenID Connect Server to provide the associated id, access and refresh tokens. 

This is done by sending a token request to the NHS Digital token endpoint as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#token-request).

The token endpoint can be obtained from the configuration data. From the example data above this would be:

```
https://connect.iam.spine2.ncrs.nhs.uk/token
``` 

An example of a request is given below.

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
  client_assertion=eyJ0eXAiOiJqd3QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjEwMCJ9.ewogICJpc3MiOiAiYWJjMTIzIiwKICAic3Vi
   IjogImFiYzEyMyIsCiAgImF1ZCI6ICJodHRwczovL2Nvbm5lY3QuaWFtLnNwaW5lMi5uY3JzLm5ocy51ay90b2tlb
   iIsCiAgImlhdCI6IDE1MDQ2MTExMzUsCiAgImV4cCI6IDE1MDQ2MTExNjUsCiAgImp0aSI6ICJqYXNkaDg3amU5cD
   hyc2RqMjMzOTIzODk4MjMiCn0. ...
```
The example shows that the request contains the client_id and redirect_uri from the original authentication request and the authorization code returned in response (for brevity the client assertion JWT value has been truncated).

### Client Authentication

The NHS Digital implementation requires that the client authenticates using the private_key_jwt method using the RSA256 algorithm. 

To achieve this the third party must include client_id, client_assertion_type and client_assertion parameters in the token request.

* The client_id parameter should be set to the value allocated to the the third party during registration and must match that in the authentication request e.g. abc123. 
* The client_assertion parameter must be set to the value urn:ietf:params:oauth:client-assertion-type:jwt-bearer. 
* The client_assertion parameter must contain a client assertion JWT signed with one of the third party's private keys.

The client assertion JWT is comprised of three Base64URL encoded elements separated by a . character. 

#### Header

The first element of the JWT is the token header. If we decode the value from the example above we get the string below:

```
{"typ":"jwt","alg":"RS256","kid":"100"}
```

This specifies that the token has been signed with an RSA Signature utilising the SHA-256 hashing algorithm and the key identified by the string “100”. This kid should relate to a key contained in the third party's JWKS returned from the third party's JWKS endpoint (see [Third Party Keys](develop_nhs_impl#third-party-keys)).

#### JSON Object

The second element gives the JSON object containing the claims about the third party application e.g. the client identifier. Decoding the the value from the example above we get the string below:

```
{
  "iss": "abc123",
  "sub": "abc123",
  "aud": "https://connect.iam.spine2.ncrs.nhs.uk/token",
  "iat": 1504611135,
  "exp": 1504611165,
  "jti": "jasdh87je9p8rsdj23392389823"
}
```

The NHS Digital implementation expects the following claims in the client assertion JSON:

|Name|Description|
|----|-----------|
|iss|This must contain the client_id of the third party application e.g. abc123.|
|sub|This must contain the client_id of the third party application e.g. abc123.|
|aud|This must contain the URL of the NHS Digital token endpoint e.g. https://connect.iam.spine2.ncrs.nhs.uk/token|
|iat|This must contain the time at which the JWT was issued represented as the number of seconds from 1970-01-01T0:0:0Z as measured in UTC (the NHS Digital implementation will check that this was in the last 40 seconds).|
|exp|This must contain the expiration time on or after which the token must not be accepted for processing.|
|jti|A unique identifier for the token, which can be used to prevent reuse of the token. The NHS Digital implementation will check that this jti value has not been used before.|

#### Signature

The third and final element of the JWT is the signature. The NHS Digital implementation requires that the token is signed with an RSA Signature utilising the SHA-256 hashing algorithm and a key whose public element is made available via the third party's JWKS endpoint.

For further information on the creation of a signed JWT see the [JSON Web Signature Specification](https://tools.ietf.org/html/rfc7515).

### Token Successful Response

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#token-successful-response).

#### ID Token

An example decoded ID Token as returned from the NHS Digital implementation is given below:

```
{
 "aud":"abc123",
 "s_hash":"OKuG1HdMk6FzQaF3WEi-VQ",
 "c_hash":"opQhesA-we--QHsXuic9TA",
 "exp":1497992399,
 "iss":"https://connect.iam.spine2.ncrs.nhs.uk/token",
 "iat":1497956399,
 "at_hash":"SjUTS24aWk6lg_i5LGATNQ",
 "amr":["pin","legacy_signed_challenge"],
 "device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ",
 "jti":"2683c14bbb7a43e3b47f1490fa448075",
 "name":"Smith John Mr",
 "sub":"uid=240000109896,ou=People,o=nhs",
 "role": {
   "name":"\\"Admin and Clerical\\":\\"Admin and Clerical\\":\\"Privacy Officer\\"",
   "role_id":"240000117896",
   "code":"S8002:G8003:R0001"},
 "org": {
   "name":"GREATER MANCHESTER STRATEGIC HA",
   "code":"Q14"},
 "activities": ["B0263","B0264"],
 "workgroups": [{
   "name": "Work Group 1",
   "code": "150210428105"},{
   "name": "Work Group 2",
   "code": "150210428106"}]
}
```

The full set of claims that may be returned in the id token are described below. In the descriptions that follow the reader is assumed to be familiar with the Spine Role Based Access Control (RBAC) and the Spine Directory, if not further information can be found in the Spine External Interface Specification (EIS).

|Name|Description|
|----|-----------|
|aud|Set to the client_id of the third party requesting the token e.g. abc123.|
|s_hash|Set to the base64url encoding of the left-most half of the SHA256 hash of the associated SSO Token. It may be ignored by the third party application.|
|c_hash|Set to the base64url encoding of the left-most half of the SHA256 hash the associated authorisation code value. Third party applications may use this to protect against token substitution attacks by checking the value against the authorization code used in the get token request.|
|exp|Set to the current date/time (in seconds from 1970-01-01T0:0:0Z as measured in UTC) plus 36,000 seconds. Ten hours represents the length of a standard Spine smartcard session.|
|iss|Set to the identity of the issuer. For the example configuration given above this would be https://connect.iam.spine2.ncrs.nhs.uk/token|
|iat|The time of issuance (in seconds from 1970-01-01T0:0:0Z as measured in UTC at the time of issuance).|
|at_hash|Set to the base64url encoding of the left-most half of the SHA256 hash of the associated access_token value. Third party applications may use this to protect against token substitution attacks.|
|amr|This defines the authentication mechanism and is currently set to the hard-coded array of values pin and legacy_signed_challenge.|
|device_id|A unique identifier for the instance of the Identity Agent Bridge through which the authentication request was made. This is set to a random value. It may be ignored by the third party application.|
|jti|An identifier for the id token. Set to a random value. It may be ignored by the third party application.|
|name|The authenticated user's name. Set to the user's Common Name (CN) as retrieved from the Spine directory e.g. Smith John Mr.|
|sub|The authenticated user's identity. Set to the user's Distinguished Name (DN) as retrieved from the directory e.g. uid=240000109896,ou=People,o=nhs.|
|role|This claim represents the default role selected by the user at the time of authentication. It comprises three elements read from the associated nhsOrgPersonRole object in the directory:|
||*role_id* - The unique identifier of the nhsOrgPersonRole object.|
||*code* - The 3-part role code taken from the nhsJobRoleCode attribute e.g. S8002:G8003:R0001.|
||*name* - The 3-part role name taken from the nhsJobRole attribute e.g. "Admin and Clerical":"Admin and Clerical":"Privacy Officer".|
|org|This claim represents the organisation to which the user's default role belongs. It comprises two elements read from the associated nhsOrgPerson object in the directory:|
||*code* - The id of the organisation taken from the nhsIDCode attribute e.g. Q14.|
||*name* - The name of the organisation taken from the o attribute e.g. GREATER MANCHESTER STRATEGIC HA.|
|activities|This claim is an array of activity codes associated with the default role taken from the nhsBusinessFunctionsCodes multi-valued attribute of the nhsOrgPersonRole object e.g. B0263,B0264.|
|workgroups|This claim is an array of workgroups to which the user's default role has been assigned. Each value comprises two elements taken from the associated nhsOrgPersonRole object in the directory:|
||*code* - The ID of the workgroup taken from the multi-valued nhsWorkGroupsCodes attribute.|
||*name* - The name of the workgroup taken from the multi-valued nhsWorkGroups attribute.|

{% include warning.html content="In the current implementation the nth workgroups code is not guaranteed to correspond to the nth workgroup name. This may be remedied in future or the workgroups element may me split into two separate elements." %}

#### ID Token Validation

The third party application should validate the id token as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#id-token-validation).

To authenticate the source of the id token the third party application should check the signature of the JWT using the algorithm and key identifier specified in the JWT header. For the NHS Digital implementation the RSA256 algorithm will have been specified and the kid will identify a key in the NHS Digital JWKS as described in the [NHS Digital Keys](develop_nhs_impl#nhs-digital-keys section].

#### Access Token

A successful token response will include an access token but as the current NHS Digital implementation does not support a userinfo endpoint this may be ignored by third party applications. 

#### Refresh Token

A successful token response will include a refresh token but as the current NHS Digital implementation does not support a userinfo endpoint this may be ignored by third party applications. 

### Token Error Response

This is as described for the Authorization Code Flow previously detailed [here](explore_auth_code_flow#token-error-response).

The NHS Digital implementation only supports the error, an error_description parameter will not be provided.

An example response from the NHS Digital implementation is given below:

```
HTTP/1.1 400 Bad Request
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "error": "invalid_request"
  }
```

The NHS Digital implementation supports the follwoing HTTP response and error combinations.

|HTTP Response|Error|
|-------------|-----|
|400|invalid_request|
|400|invalid_grant|
|401|invalid_client|
|500|internal_server_error|
