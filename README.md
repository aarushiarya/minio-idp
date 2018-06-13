# doctest
# OpenID Connect Minio Deployment Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(Keycloak), add host to minio server using access_token and Idp.

## Example 1
### Add application to Keycloak

1. Download and install [keycloak](https://www.keycloak.org/docs/latest/getting_started/index.html)
2. Follow the instructions to make your admin console. Then create a realm, add you application in Clients. 
3. Under Clients, go to settings, select Access type as Confidential. Save the changes.
4. Go to Credentials on top, configure this Client Id and Client Secret in your application to request access token from Idp(Keycloak).

![image](https://user-images.githubusercontent.com/22103395/41311551-6ce39d44-6e39-11e8-89c8-9c8686e58bdb.png)

### Login to Minio 
Login with Minio using access token and Idp (Keycloak).

```
add new host access_token Idp
```

### Minio using public key from Idp to verify access_token

Minio server verifies access_token and if successful, returns temporary access key and secret key. Otherwise, give an error of invalid access_token.

Client application can use Minio resources.

## Example 2
### Add application to wso2

1. Download and install [wso2](https://docs.wso2.com/display/IS530/Installation+Guide)
2. Register your application on the [management console](https://docs.wso2.com/display/IS530/Getting+Started+with+the+Management+Console)
3. Follow this [tutorial](https://docs.wso2.com/display/IS530/Setting+Up+the+Sample+Webapp) to setup your first application.
4. Add user to get access token.


