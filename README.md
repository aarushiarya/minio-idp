
# Minio Identity Management Configuration Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(wso2).

## How OpenID works?

![image](https://user-images.githubusercontent.com/22103395/41444342-662c7a42-6ff7-11e8-93aa-75bce207e6cd.png)

## 1. Client request for access_token from wso2 by sending client id and client secret.

   - Request
```
curl -v -X POST -H "Authorization: Basic <base64 encoded client id:client secret value>" -k -d "grant_type=client_credentials" -H "Content-Type:application/x-www-form-urlencoded" https://localhost:9443/oauth2/token
```

## 2. wso2 returns access_token.

  - Response
```
{"token_type":"Bearer","expires_in":2061,"access_token":"ca19a540f544777860e44e75f605d927"}

```

...Client credentials grant in wso2 don't support refresh token. If access_token expires, make request to wso2 to get new ...access_token and get new temporary credentials from minio.

## 3. Client request for temporary credentials by making POST request to Minio with the access_token.

## 4. Minio verify access_token by making a POST request to OAuth Introspection Endpoint using username and password.

  - Request 
```
curl -k -u <USERNAME>:<PASSWORD> -H 'Content-Type: application/x-www-form-urlencoded' -X POST --data 'token=<ACCESS_TOKEN>' https://localhost:9443/oauth2/introspect
```

...Here USERNAME can be of any user with "/permission/admin/manage/identity/applicationmgt/view" permissions. For more information, refer to the documentation [here](https://docs.wso2.com/display/IS530/Invoke+the+OAuth+Introspection+Endpoint).

## 5. wso2 returns a response whether the token is valid or not. If valid, responds with the expiry time of access_token

  - Response
```
{"exp":1464161608,"username":"admin@carbon.super","active":true,"token_type":"Bearer","client_id":"rgfKVdnMQnJSSr_pKFTxj3apiwYa","iat":1464158008}
```

## 6. If access_token is valid, Minio returns temporary credentials comprising of Access key, Secret key and Session token and expiry time.


## Configuring Identity Provider with Client Grants
Client Credentials Grant is suitable for machine-to-machine authentication or for a client making requests to an API that does not require the user’s permission. To read more, go to the [documentation](https://docs.wso2.com/display/IS510/Client+Credentials+Grant).

#### Configure Identity Provider wso2
  - Download and install [wso2](https://docs.wso2.com/display/IS530/Installation+Guide)
  - Register your application on the [management console](https://docs.wso2.com/display/IS530/Getting+Started+with+the+Management+Console)
  - Follow this [tutorial](https://docs.wso2.com/display/IS530/Setting+Up+the+Sample+Webapp) to setup your first application.
  - Add user to get access token. 

## Setting Identity Provider Information in Minio Server
#### Changes in Minio


## Example Code for Clients
