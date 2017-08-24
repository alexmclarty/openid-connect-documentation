---
title: Spine OpenID Connect Implementation
keywords: explore, design, reference
tags: [design,explore]
sidebar: overview_sidebar
permalink: explore_spine_implementation.html
summary: An overview of the Spine implementation of OpenID Connect.
---

{% include important.html content="Work in Progress." %}

## Architecture

The diagram below depicts at a a high level the Authorization Code Flow.

![Spine OpenID Connect Implementation](/images/OIDCSpineImplementation.jpg)

## Configuration Endpoint

https://connect.iam.spine2.ncrs.nhs.uk:4443/.well-known/openid-configuration

```
{"token_endpoint": "https://connect.iam.spine2.ncrs.nhs.uk:4444/token", "scopes_supported": ["openid"], "id_token_signing_alg_values_supported": ["RS256"], "subject_types_supported": ["public"], "token_endpoint_auth_methods_supported": ["private_key_jwt"], "grant_types_supported": ["authorization_code"], "response_types_supported": ["code"], "issuer": "https://connect.iam.spine2.ncrs.nhs.uk/token", "authorization_endpoint": "https://connect.iam.spine2.ncrs.nhs.uk:4443/authorize", "claims_supported": ["sub", "iss", "name", "org", "role", "workgroups", "activities"], "request_uri_parameter_supported": false, "token_endpoint_auth_signing_alg_values_supported": ["RS256"], "jwks_uri": "https://connect.iam.spine2.ncrs.nhs.uk:4443/.well-known/jwks.json"}
```

## JWKS Endpoint

```
https://connect.iam.spine2.ncrs.nhs.uk:4443/.well-known/jwks.json
{"keys": [{"kid": "rF-Do8xhajL-bfrC0Ef0DpqvwbJ9UUzUbEFQwl42rw4", "e": "AQAB", "kty": "RSA", "n": "vsqnkkTNOlMC9boScmBCFXQ2BpQwdOGd3ET55PY4wBzZ217opPqiP43_yZBgWfG3Mb1r3PaDNCtAZSbsWH_iIvLZnllnvzYY_iOW7zrqaWSD4Dfaaa2Eg4dOBI71n4A38FRsa2N1nbWKCdX_ffZwqxZwX0dg44ELvwO5rTwJLoIZPXy2FD_lkjrR8ksC_N9EiQ-yY9bbQTIgVZuHXdklZnTTCeUczz7bAt3DVJ5JFOT1E7sBkUqOqwXLG7k-capqWYJXUNgKFig0dt8AEPPvAhXsnt4fKOQQx2Kby-YNYijpxP752P62ArGLaqcvv5hU13V2obPMQ4P9ysHBFyEurw"}]}
```

## ID Token

```
{"role":"S0080:G0450:R5080","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","jti":"0cca2fe5675f4fe28f687ec5a2dac187","activities":["B0005","B0008"],"exp":1503584507}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - Refresh Token Decoded: {"iv":"LopflBFmvq3BJRUm5r2v3cGpVE7iEyLN4Qf8YLvXS4o=","role_id":"240000115894","device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","sub":"uid=240000109896,ou=People,o=nhs","iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"https://connect.iam.spine2.ncrs.nhs.uk/token","sso_ticket":"3zkYKhNMgbFwVqAy0aBS8W93epK8k4xsBWP-8sbns0SJmr1z2B5hFJHbe8txQQ2eluxmULLB2ogxHYpZvyQ=","jti":"2617f03c0b44454b93b427c5d77e704d","tag":"dkQryHQjQ1KrYlFDaNz_3Q==","exp":1503620477}
2017-08-24 15:21:12,663 INFO test_get_tokens.py.test__get_tokens() - ID Token Decoded: {"device_id":"AMuvc7H9TlriCn1Rwjz5jWUAVvAqhymJ","role":{"role_id":"240000115894","name":"\"Admin & Clerical\":\"Management - A & C\":\"Registration Authority Manager\"","code":"S0080:G0450:R5080"},"activities":["B0005","B0008"],"amr":["pin","legacy_signed_challenge"],"jti":"2b44cf3602d445e0909a808ad3194341","name":"Seven User Mr","s_hash":"LLJ6GrrF3GenakaKAyYBTg","at_hash":"gNn8ijUTnwM34MxSFzBgYw","sub":"uid=240000109896,ou=People,o=nhs","org":{"name":"GREATER MANCHESTER STRATEGIC HA","code":"Q14"},"iat":1503584477,"iss":"https://connect.iam.spine2.ncrs.nhs.uk/token","aud":"s6BhdRkqt3","exp":1503620477,"c_hash":"MPZJ1jWGHrGqL-fMHAEBMQ"}
```
