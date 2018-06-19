
# Minio Identity Management Configuration Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(WSO2).

## Minio - WSO2 Flow
 
![image](https://user-images.githubusercontent.com/22103395/41444342-662c7a42-6ff7-11e8-93aa-75bce207e6cd.png)

### 1. Client requests for access_token from WSO2 by sending client id and client secret.

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
curl -v -X POST -H "flow:client_credentials" -k -d "token=<ACCESS_TOKEN>" -H "Content-Type:application/x-www-form-urlencoded" https://localhost:9000/minio/sts
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
  - Accept access_token
  ```
  
  type credentials struct {
	AccessKey string `json:"accessKey,omitempty"`
	SecretKey string `json:"secretKey,omitempty"`
	ExpTime   float64
  }

  func get_access_token(){
	// Create a variable with wso2 application details after registering application on wso2 IDP
	// Import "golang.org/x/oauth2/clientcredentials"
	var (
		oauthConf = &clientcredentials.Config{
			ClientID:     "<Application Client ID>",
			ClientSecret: "<Application Client Secret>",
			TokenURL:     "<wso2 token endpoint>", // Example: https://localhost:9443/oauth2/token"
			Scopes:       []string{"openid", "profile", "email", "offline_access"},
		}
		// random string for oauth2 API calls to protect against CSRF
		oauthStateString = "thisshouldberandom"
	)

	// enable if HTTPS connection is not secure
	//http.DefaultTransport.(*http.Transport).TLSClientConfig = &tls.Config{InsecureSkipVerify: true}
	//

	// Make an authorization request to wso2 access token endpoint 
	accessToken, err := oauthConf.Token(oauth2.NoContext)
	if err != nil {
		log.Fatalln("Error In Token Request")
		return "", err
	}
	
	// Make a POST request to Minio server with access token obtained
	minioTokenUrl := <Insert the URL> //Example "http://localhost:9000"
	resource := "/minio/sts" //Endpoint within minio server

	u, err := url.ParseRequestURI(minioTokenUrl)
	if err != nil {
		log.Fatalln(err)
	}

	u.Path = resource
	urlStr := u.String()
	data := url.Values{}
	data.Add("AccessToken", accessToken) //Access token is added to the body of the POST Request to Minio Server

	// Create an HTTP client to make post request
	client := &http.Client{}
 
	r, err := http.NewRequest("POST", urlStr, strings.NewReader(data.Encode())) //bytes.NewBufferString(data.Encode())
	if err != nil {
		//return nil, err
		log.Fatalln(err)
	}
	
	//Add headers to POST request
	r.Header.Add("Content-Type", "application/x-www-form-urlencoded")
	
	//Execute POST request 
	resp, err := client.Do(r)
	if err != nil {
		log.Fatalln(err)
	}
	
	//Capture response in Body
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		//return nil, err
		log.Fatalln(err)
	}
	
	// Minio Server will return Minio Secret Key, Minio Access Key, Expiration time of validated Access Token  
	cred := credentials{}	
	json.Unmarshal(body, cred)
	defer resp.Body.Close()
	return cred, nil
  }
  ```
  
  - Issue new temporary credentials
  ```
  func get_new_cred(){
	c, err := auth.GetNewCredentials()
	if err != nil {
		log.Fatalln(err)
	}
  }
  ```
