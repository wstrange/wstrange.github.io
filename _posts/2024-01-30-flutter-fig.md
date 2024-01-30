---
title: gRPC authentication and Flutter
description: gRPC authentication and Flutter
tags:
- Flutter
- Dart
- gRPC

---

# gRPC authentication and Flutter

I have always been a fan of the "contract first" approach that gRPC and protobuf provides. There 
are plenty of tutorials that explain how to use gRPC with Dart, but they mostly gloss over
the problem of authentication and session management. 

To that end, I created a proof of concept on how to implement gRPC authentication between
a Flutter client and a Dart gRPC service. 

The [Fig](https://github.com/wstrange/fig) provides server (`fig_auth`) and client (`fig_flutter`) libraries to 
provide authentication and session management using gRPC. 


The basic gist of Fig (Firebase Identity for Flutter using gRPC) is this:

* Your Flutter client authenticates to Firebase and obtains an OIDC jwt token.
* The client calls the `Authenticate` gRPC method provided by `fig_auth`, passing along the OIDC token.
* The `fig_auth` package validates the OIDC token using PKI.
* If the token is valid, fig_auth creates a `Session` for the user, and returns an opaque session cookie
to the client in the response.
* The client provides the session cookie in subsequent calls using a gRPC `Authorization` header. This
 is injected into gRPC calls by a client interceptor. 
* The service interceptor provided by `fig_auth` looks for a valid session cookie. If the cookie is valid, and 
 the session has not timed out, the call will be allowed to proceed to your gRPC method. If the
 session is not valid, or a cookie is not provided, a gRPC error will be returned to the client.
* You can mark some of your gRPC methods as being _unauthenticated_. The interceptor will not enforce
 the previous rules.



