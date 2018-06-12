# doctest
# OpenID Connect Minio Deployment Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(Keycloak), add host to minio server using access_token and Idp.

## Example  
### Add application to Keycloak

1. Download and install keycloak https://www.keycloak.org/docs/latest/getting_started/index.html
2. Follow the instructions to make your admin console. Then create a realm, add you application in Clients. 
3. Under Clients, go to settings, select Access type as Confidential. Save the changes.
4. Go to Credentials on top, configure this Client Id and Client Secret in your application to request access token from Idp(Keycloak).

![Screenshot](keycloak.jpeg)

### Login to Minio 
Login with Minio using access token and Idp (Keycloak).

```
add new host access_token Idp
```

### Minio using public key from Idp to verify access_token

Minio server verifies access_token and if successful, returns temporary access key and secret key. Otherwise, give an error of invalid access_token.

Client application can use Minio resources.


