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
- **refresh-path** string
  The path to refresh a token. Example: **/refresh**
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
  "token": "{token}",
  "expires-in": "{expires-in}",
  "refresh-token": "{refresh-token}"
}
```

The JSON data have following properties:
- **host** string  
  Sender host
- **token** string  
  A token to be used by the destination host to send a packet.
- **expires-in** integer  
  Seconds
- **refresh-token**  
  A token to be used by the destination host to refresh the token.

The destination host:
- MUST always returns a 204 response.
- MUST NOT use the *token* and *refresh-token* yet.
- MUST make a request to grant a token request.


### Grant a token request

A sender host makes the following request with any other headers over TLS:
```
POST {grant-path} HTTP/1.1
Host: {host}
Authorization: Bearer {token}
Content-Type: application/json

{
  "host": "{sender host}",
  "token": "{token}",
  "expires-in": "{expires-in}",
  "refresh-token": "{refresh-token}"
}
```

The JSON data have following properties:
- **host** string  
  Sender host
- **token** string  
  A generated token to be used by the destination host to send a packet.
- **expires-in** integer  
  Seconds
- **refresh-token**  
  A token to be used by the destination host to refresh the token.

The destination host:
- MUST returns a 204 response if the request is valid.
- MUST returns an error response if the request is invalid.
- can use the *token* and *refresh-token* if the request is valid.

The sender host:
- can use the *token* and *refresh-token* given by the token request if the response is 204.

### Refresh a token

A sender host makes the following request with any other headers over TLS:
```
POST {refresh-path} HTTP/1.1
Host: {host}
Authorization: Bearer {token}
```

The destination host:
- MUST returns the following response with any other headers if the request is valid:
  ```
  HTTP/1.0 200 OK
  Content-Type: application/json

  {
    "token": "{token}",
    "expires-in": "{expires-in}",
    "refresh-token": "{refresh-token}"
  }
  ```

  The JSON data have following properties:
  - **token** string  
    A new token.
  - **expires-in** integer  
    Seconds
  - **refresh-token**  
    A new refresh token.
- MUST returns an error response if the request is invalid.

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
