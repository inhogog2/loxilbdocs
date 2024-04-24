# API server
Generic library for building a LoxiLB API server.

# Usage
현재 API 서버는 HTTP, HTTPS모두 지원하며, Loxilb 실행시 Default로 실행됩니다.

API에 사용되는 옵션은 다음과 같습니다. HTTPS에 대한 실행은 후술될 내용을 참조하시길 바랍니다.

The current API server supports both HTTP and HTTPS. It runs as default when running Loxilb.
The options used in the API are as follows. For running HTTPS, please refer to the following sections.

```
      --host=            the IP to listen on (default: localhost) [$HOST]
      --port=            the port to listen on for insecure connections, defaults to a random value [$PORT]
      --tls              enable TLS  [$TLS]
      --tls-host=        the IP to listen on for tls, when not specified it's the same as --host [$TLS_HOST]
      --tls-port=        the port to listen on for secure connections, defaults to a random value [$TLS_PORT]
      --tls-certificate= the certificate to use for secure connections [$TLS_CERTIFICATE]
      --tls-key=         the private key to use for secure connections [$TLS_PRIVATE_KEY]
```

실제 사용하는 예시는 다음과 같습니다.

Examples of practical use are as follows.

```
 ./loxilb --tls-key=api/certification/server.key --tls-certificate=api/certification/server.crt --host=0.0.0.0 --port=11111 --tls-port=8091
```

# API list
예시로 보여드릴 API는 Load balancer 에 대한 Create, Delete, Read API가 있습니다.

For example, the API has Create, Delete, and Read APIs for Load balancer.

| Method | URL | Role | 
|------|---|---|
| GET|/netlox/v1/config/loadbalancer/all | Get the load balancer information |
| POST|/netlox/v1/config/loadbalancer| Add the load balancer information to LoxiLB |
| DELETE|/netlox/v1/config/loadbalancer/externalipaddress/{IPaddress}/port/{#Port}/protocol/{protocol} | Delete the load balacer infomation from LoxiLB|

더 자세한 정보(Param, Body 등)은 Swagger문서를 참조 하시길 바랍니다.

See Swagger documentation for more information (Param, Body, etc.).

# HTTPS guide
 Key and Cert files are required for HTTPS, and they are not detailed, but explain how to generate them and where LoxiLB can read and use user-generated Key and Cert files.

```
      --tls              enable TLS  [$TLS]
      --tls-host=        the IP to listen on for tls, when not specified it's the same as --host [$TLS_HOST]
      --tls-port=        the port to listen on for secure connections (default: 8091) [$TLS_PORT]
      --tls-certificate= the certificate to use for secure connections (default:
                         /opt/loxilb/cert/server.crt) [$TLS_CERTIFICATE]
      --tls-key=         the private key to use for secure connections (default:
                         /opt/loxilb/cert/server.key) [$TLS_PRIVATE_KEY]
```
To enable https on LoxiLB, we changed it to enable it using the  `--tls`option. 

Tls-host and tls-port are the contents of deciding which IP to listen to. The default IP address used as tls-host is 0.0.0.0, which is everywhere, but for future security, we recommend doing only certain values. The port is 8091 as the default. You can also find and change this from a value that does not overlap with the service you use.

LoxiLB reads the key by default as /opt/loxilb/cert/path with server.key and the Cert file as server.crt in the same path. In this article, we will learn how to create the server.key and server.crt files.

## Preparation
First of all, the simplest way is to create it using *openssl*. To install openssl, you can install it using the command below.
```
apt install openssl
```
The LoxiLB team confirmed that it operates on 1.1.1f version of openssl.
```
openssl version
OpenSSL 1.1.1f  31 Mar 2020
```
### 1. Create server.key 

```bash
openssl genrsa -out server.key 2048
```

The way to generate server.key is simple. You can create a new key by typing the command above. In fact, if you type in the command, you can see that the process is output and the server.key is generated.
```bash
openssl genrsa -out server.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..............................................+++++
...........................................+++++
e is 65537 (0x010001)
```

### 2. Create server.csr 

```bash
openssl req -new -key server.key -out server.csr
```

Create a csr file by putting the desired value in the corresponding item. This file is not used directly for https, but it is necessary to create a Cert file to be created later. When you type in the command above, a long sentence appears asking you to enter information, and you can fill in the corresponding value according to your situation.
```bash
openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

### 3. Create server.crt
```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key  -out server.crt
```
This is the process of creating server.crt using server.key and server.csr generated above. You can issue a certificate with a limited deadline by setting the expiration date of the certificate well and putting a value after -day. The server.crt file is created with the following output.
```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key  -out server.crt
Signature ok
subject=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
Getting Private key
```

### 4. Validation
You can enable https with the server.key and server.cert files generated through the above process.

If you move all of these files to the `/opt/loxilb` path and check them, you can see that they work well.

```bash
sudo cp server.key /opt/loxilb/cert/.
sudo cp server.crt /opt/loxilb/cert/.
```

```bash
 curl http://0.0.0.0:11111/netlox/v1/config/loadbalancer/all
{"lbAttr":[]}

 curl -k https://0.0.0.0:8091/netlox/v1/config/loadbalancer/all
{"lbAttr":[]}
```

It should appear in the log as follows.

```bash
2024/04/12 16:19:48 Serving loxilb rest API at http://[::]:11111
2024/04/12 16:19:48 Serving loxilb rest API at https://[::]:8091
```

# LoxiLB API development guide
## API source Architecture
```
.
├── certification
│   ├── serverca.crt
│   └── serverkey.pem
├── cmd
│   └── loxilb_rest_api-server
│       └── main.go
├── ….
├── models
│   ├── error.go
│   ├── …..
├── restapi
│   ├── configure_loxilb_rest_api.go
│   ├── …..
│   ├── handler
│   │   ├── common.go
│   │   └──…..
│   ├── operations
│   │   ├── get_config_conntrack_all.go
│   │   └── ….
│   └── server.go
└── swagger.yml

```
* Except for the ./api/restapi/handler and ./api/certification directories, the rest of the contents are automatically created.
* Add the logic for the function to the handler directory.
* Add logic to file ./api/restapi/configure_loxilb_rest_api.go

1. Swagger.yml file update
```
paths:
  '/additional/url/{param}':
    get:
      summary: Test Swagger API Server.
      description: Check Swagger API server. This basic information or architecture is for the later applications.
      parameters:
        - name: param
          in: path
          required: true
          type: string
          format: string
          description: Description of the additional url
      responses:
        '204':
          description: OK
        '400':
          description: Malformed arguments for API call
          schema:
            $ref: '#/definitions/Error'
        '401':
          description: Invalid authentication credentials

```
* path.{Set path and parameter URL}.{get,post,etc RESTful setting}.{Description}
- {Set path and parameter URL}
Set the path used in the RESTful API.
It begins with "config/" and is defined as a sub-category from a large category.
Define the parameters using the symbol {param}. The parameters are defined in the description section.
- {get,post,etc RESTful setting}
Use get, post, delete, and patch to define queries, registrations, deletions, and modifications.
- {Description}
Summary description of API
Detailed description of API
Parameters
Set the name, path, etc.
Define the content of the response

2. Creating Additional Parts with Swagger
```
# alias swagger='docker run --rm -it  --user $(id -u):$(id -g) -e GOPATH=$(go env GOPATH):/go -v $HOME:$HOME -w $(pwd) quay.io/goswagger/swagger'
# swagger generate server
```

3. Development of Additional Partial Handlers
```
package handler

import (
    "fmt"

    "github.com/go-openapi/runtime/middleware"

    "testswagger.com/restapi/operations"
)

func ConfigAdditionalUrl(params operations.GetAdditionalUrlParams) middleware.Responder {
    /////////////////////////////////////////////
    //             Add logic Here              //
    ////////////////////////////////////////////.
    return &ResultResponse{Result: fmt.Sprintf("params.param : %s", params.param)}
}

```
* Select the logic required for the ConfigAdditionalUrl portion of the handler directory. The required parameters come from operations.GetAdditionalUrlParams.

4. Additional Partial Handler Registration
```
func configureAPI(api *operations.LoxilbRestAPIAPI) http.Handler {
    ...... 
    // Change it by putting a function here
    api.GetAdditionalUrlHandler = operations.GetAdditionalUrlHandlerFunc(handler.ConfigAdditionalUrl)
 ….
}
```
* if api.{REST}...The Handler form is automatically generated, where if nil is erased and a handler is added to the operation function.
*    In many cases, additional generation is not possible. In that case, you can add the function by entering it separately. The name of the function consists of a combination of Method, URL, and Parameter.

