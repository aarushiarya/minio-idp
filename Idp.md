# Minio Identity Management Configuration Guide
Minio Identity Management uses `WSO2` to get tokens using client credentials grant. If token is verified, minio gives temporary
access key and secret key. These credentials are stored in `etcd`.

### 1. Prerequisites
Install minio server from [here](https://docs.minio.io/).

### 2. Install
1. Download and install [WSO2](https://docs.wso2.com/display/IS530/Installation+Guide).
2. Download and install [Etcd v3](https://github.com/coreos/etcd/releases).

### 3. Configuring 
Let's configure WSO2 and Etcd v3 to get started.

#### WSO2
1. Start WSO2 server and login to the [management console](https://docs.wso2.com/display/IS530/Getting+Started+with+the+Management+Console).
2. Add new `Realm` and `Service Provider`.
3. In Service Provider, select `Inbound Authentication Configuration` and go to `OAuth/OpenID Connect Configuration`. 
   Select `Client Credentials`, save OAuth Client Key and OAuth Client Secret.
4. Add those `Client Credentials` to the request to minio STS endpoint. Give the endpoint in config.json.
5. Minio server requests token from WSO2 and verifies the signature. 
6. If valid, Minio returns temporary credentials.

#### Etcd v3 
1. Start etcd server.

### 4. Test


### 5. Run 

