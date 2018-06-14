# doctest
# OpenID Connect Minio Deployment Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(wso2), add host to minio server using access_token and Idp.

![image](https://user-images.githubusercontent.com/22103395/41384339-07613fd4-6f2a-11e8-8815-3593342275d8.png)

1. Client request for access_token from wso2 by sending client id and client secret.

#### Request
```
curl -v -X POST -H "Authorization: Basic <base64 encoded client id:client secret value>" -k -d "grant_type=client_credentials" -H "Content-Type:application/x-www-form-urlencoded" https://localhost:9443/oauth2/token
```

2. wso2 returns access_token.

#### Response
```
{"token_type":"Bearer","expires_in":2061,"access_token":"ca19a540f544777860e44e75f605d927"}

```

3. Client request for temporary credentials by making a POST request to Minio with the access_token.

#### Request
```
POST --data 'token=<ACCESS_TOKEN>'
```

4. Minio verify access_token by making a POST request to OAuth Introspection Endpoint using username and password.

#### Request 
```
curl -k -u <USERNAME>:<PASSWORD> -H 'Content-Type: application/x-www-form-urlencoded' -X POST --data 'token=<ACCESS_TOKEN>' https://localhost:9443/oauth2/introspect
```

We can use <USERNAME>:<PASSWORD> of any user with "/permission/admin/manage/identity/applicationmgt/view" permissions. For more information, refer to the documentation [here](https://docs.wso2.com/display/IS530/Invoke+the+OAuth+Introspection+Endpoint).

5. wso2 returns a response whether the token is valid or not.

#### Response
```
{"exp":1464161608,"username":"admin@carbon.super","active":true,"token_type":"Bearer","client_id":"rgfKVdnMQnJSSr_pKFTxj3apiwYa","iat":1464158008}
```

6. If token is valid, Minio returns temporary credentials comprising of Access key, Secret key and Session token(STS).
  Temporary credentials are stored as environment variables, until they expire.
  
