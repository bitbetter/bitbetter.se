---
title: AWS Cognito JWT tokens are weird
date: 2020-04-14
tags: [AWS, Security]
draft: true
---

The HttpApi needs Audience and scopes in CF

```json
"HttpApi": {
    "Type": "AWS::Serverless::HttpApi",
    "Properties": {
      "Auth": {
        "DefaultAuthorizer": "Authorizer",
        "Authorizers": {
          "Authorizer": {
            "JwtConfiguration": {
              "Issuer": "https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_RPoiLFejX",
              "Audience": { "Ref": "AuthJwtAudiences" }
            },
            "IdentitySource": "$request.header.Authorization"
          }
        }
      }
    }
}
```

ID TOKEN:

```json
{
  "sub": "0d1679d2-7cd5-4db3-b329-481a1eacdb79",
  "aud": "3smh2sm9gk4ae640gegua7cla6",
  "email_verified": true,
  "event_id": "73348941-5106-4876-9056-9a42e4286e80",
  "token_use": "id",
  "auth_time": 1586890444,
  "iss": "https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_RPoiLFejX",
  "cognito:username": "0d1679d2-7cd5-4db3-b329-481a1eacdb79",
  "exp": 1586894044,
  "iat": 1586890444,
  "email": "martin.kretz@finch3d.com"
}
```

ACCESS TOKEN:

```json
{
  "sub": "0d1679d2-7cd5-4db3-b329-481a1eacdb79",
  "event_id": "d45713df-ec33-4911-acb7-780ce04a06e0",
  "token_use": "access",
  "scope": "aws.cognito.signin.user.admin",
  "auth_time": 1586929858,
  "iss": "https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_RPoiLFejX",
  "exp": 1586933458,
  "iat": 1586929858,
  "jti": "50856a2d-0eaa-4ee9-9201-90ebdf20ca29",
  "client_id": "3smh2sm9gk4ae640gegua7cla6",
  "username": "0d1679d2-7cd5-4db3-b329-481a1eacdb79"
}
```

However, the access token from AWS Cognito does not have `aud`. The IdToken does! However, it does not have scopes.
