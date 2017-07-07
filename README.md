# HTTPS TOKEN CONNECTION

*https-token* is a connection protocol for YMP.

## Name

https-token


## Version

0.0.0


## Parameters

*parameters* has following properties:
- **request-path** string
  The path to request a token. Example: **/request**
- **grant-path** string
  The path to grant a token request. Example: **/grant**
- **packet-path** string
  The path to send a packet. Example: **/packet**


## Specification

- HTTP/1.1 and HTTP/2.0 can be used.
- 

### Request a token

A sender host makes the following request with any other headers over TLS:
```
POST {request-path} HTTP/1.1
Host: {host}
Content-Type: application/json

{
  "host": "{sender host}",
  "state": "{state}"
}
```

The JSON data have following properties:
- **host** string  
  The sender host
- **state** string  
  A random string to verify a request to grant a token request.

The destination host:
- MUST always returns a 204 response.
- MUST make a request to grant a token request with the given *state*.


### Grant a token request

A sender host makes the following request with any other headers over TLS:
```
POST {grant-path} HTTP/1.1
Host: {host}
Content-Type: application/json

{
  "host": "{sender host}",
  "token": "{token}",
  "expires": "{expires}",
  "state": "{state}"
}
```

The JSON data have following properties:
- **host** string  
  The sender host
- **token** string  
  A generated token to be used by the destination host to send a packet.
- **expires** integer  
  A number of seconds from 1970-01-01T00:00:00Z without applying leap seconds.
- **state** string  
  The string given by the token request.

The destination host:
- MUST verify the *state*.
- MUST returns a 204 response if the request is valid.
- MUST returns an error response if the request is invalid.
- can use the *token* to send packets if the request is valid.

The sender host:
- MAY revoke the token if the response is not a 204.

### Send a packet

A sender host makes the following request with any other headers over TLS:
```
POST {packet-path} HTTP/1.1
Host: {host}
Authorization: Bearer {token}
Content-Type: application/json

{
  "packet": {
    "messages": [...]
  }
}
```

The JSON data have following properties:
- **packet** object
  A YMP packet

The destination host:
- MUST returns a 204 response if the request is valid.
- MUST returns an error response if the request is invalid.
