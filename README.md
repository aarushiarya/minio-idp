
# Minio Identity Management Configuration Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(WSO2).

## Minio - WSO2 Flow
 
![image](https://user-images.githubusercontent.com/22103395/41629299-93d3abac-73dd-11e8-9c88-948db2cc51e7.png)

### 1. Client requests for access_token from WSO2 by sending client id and client secret using client credentials grant.

   - Request
```
curl -v -X POST -H "Authorization: Basic <base64 encoded client id:client secret value>" -k -d "grant_type=client_credentials" -H "Content-Type:application/x-www-form-urlencoded" https://localhost:9443/oauth2/token
```

### 2. WSO2 returns access_token.

  - Response
```
{"token_type":"Bearer","expires_in":2061,"access_token":"ca19a540f544777860e44e75f605d927"}

```

Client credentials grant in WSO2 don't support refresh token. If access_token expires, make request to WSO2 to get new access_token and get new temporary credentials from minio.

### 3. Client request for temporary credentials by making POST request to Minio with the access_token.
- Proposed Request
```
curl -X POST -k "{token=<ACCESS_TOKEN>}" -H "Content-Type:application/json" https://minio-server.com:9000/minio/sts
```

### 4. Minio verify access_token by making a POST request to OAuth Introspection Endpoint using username and password.

  - Request 
```
curl -k -u <USERNAME>:<PASSWORD> -H 'Content-Type: application/x-www-form-urlencoded' -X POST --data 'token=<ACCESS_TOKEN>' https://localhost:9443/oauth2/introspect
```

Here USERNAME can be of any user with "/permission/admin/manage/identity/applicationmgt/view" permissions. For more information, refer to the documentation [here](https://docs.wso2.com/display/IS530/Invoke+the+OAuth+Introspection+Endpoint).

### 5. WSO2 returns a response whether the token is valid or not. If valid, responds with the expiry time of access_token

  - Response
```
{"exp":1464161608,"username":"admin@carbon.super","active":true,"token_type":"Bearer","client_id":"rgfKVdnMQnJSSr_pKFTxj3apiwYa","iat":1464158008}
```

### 6. If access_token is valid, Minio returns temporary credentials comprising of Access key, Secret key and Session token and expiry time.


## Configuring Identity Provider with Client Grants
Client Credentials Grant is suitable for machine-to-machine authentication or for a client making requests to an API that does not require the user’s permission. To read more, go to the [documentation](https://docs.wso2.com/display/IS510/Client+Credentials+Grant).

#### Configure Identity Provider WSO2
  - Download and install [WSO2](https://docs.wso2.com/display/IS530/Installation+Guide)
  - Register your application on the [management console](https://docs.wso2.com/display/IS530/Getting+Started+with+the+Management+Console)
  - Follow this [tutorial](https://docs.wso2.com/display/IS530/Setting+Up+the+Sample+Webapp) to setup your first application.
  - Add user to get access token. 

## Setting Identity Provider Information in Minio Server
#### Changes in Minio Server
  - Minio server accepts access_token from the client.
  - Verify if access_token is valid.
  - If valid, give new temporary credentials to the client.

## Example Code for Clients
 ```
 import (
    "encoding/json"
    "fmt"
    "net/http"
    "strings"
    "time"

    "golang.org/x/oauth2"
    "golang.org/x/oauth2/clientcredentials"
)

type minioTempCredentials struct {
    AccessKey    string    `json:"accessKey"`
    SecretKey    string    `json:"secretKey"`
    SessionToken string    `json:"sessionToken"`
    Expiration   time.Time `json:"expiration"`
}

func getMinioTempCredentials() (minioTempCredentials, error) {
    cred := credentials{}

    // Create a variable with wso2 application details after registering application on wso2 IDP
    oauthConf := &clientcredentials.Config{
        ClientID:     "<Application Client ID>",
        ClientSecret: "<Application Client Secret>",
        TokenURL:     "https://wso2-server:9443/oauth2/token",
    }

    // Make an authorization request to wso2 access token endpoint
    token, err := oauthConf.Token(oauth2.NoContext)
    if err != nil {
        return cred, err
    }

    postBody := fmt.Sprintf("{accessToken:%s}", token.AccessToken)
    // Make a POST request to Minio server with access token obtained
    resp, err := http.Post("http://minio-server.com/minio/sts", "application/json", strings.NewReader(postBody))
    if err != nil {
        return cred, err
    }
    defer resp.Body.Close()

    // Minio Server will return Minio Secret Key, Minio Access Key, Expiration time of validated Access Token
    err = json.NewDecoder(resp.Body).Decode(&cred)
    if err != nil {
        return cred, err
    }
    return cred, nil
}
```
