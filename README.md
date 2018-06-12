# doctest
OpenID Connect Minio Deployment Guide

To authenticate applications register your application with the Identity Provider(), add host to minio server using access_token and Idp.

Example - Add application to Keycloak

Download and install keycloak https://www.keycloak.org/docs/latest/getting_started/index.html
Follow the instructions to make your admin console. Then create a realm, add you application in Clients. 
Under Clients, go to settings, select Access type as Confidential. Save the changes.
Go to Credentials on top, configure this Client Id and Client Secret to request access token from Idp.
Login with Minio passing access token and Idp (Keycloak).


