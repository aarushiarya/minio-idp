# doctest
# OpenID Connect Minio Deployment Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(Keycloak), add host to minio server using access_token and Idp.

![image](https://user-images.githubusercontent.com/22103395/41384339-07613fd4-6f2a-11e8-8815-3593342275d8.png)

1. Client request for access_token from wso2 by sending clientID, client secret.
2. wso2 returns access_token
3. POST request to Minio with the access_token
4. Minio verify access_token by making a POST request to OAuth Introspection Endpoint
5. wso2 returns a response whether the token is valid or not.
6. If token is valid, Minio returns temporary credentials comprising of Access key, Secret key and STS token.
