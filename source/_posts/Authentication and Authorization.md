---
title: Authentication and Authorization
date: 2020-08-17 15:16:46
updated: 2020-08-17 16:16:46
tags: [System Design]
typora-root-url: ../
---

OAuth2 and OpenID Connect

<!-- more -->

## SSO

Single sign-on (SSO) is an authentication scheme that allows a user to log in with a single ID and password to any of several related, yet independent, software systems. It is often accomplished by using the **Lightweight Directory Access Protocol (LDAP)** and stored LDAP databases on (directory) servers.

LDAP is a protocol that defines how one should talk to a directory server. Most systems use LDAP to talk to a directory to retrieve user accounts, verify them and retrieve attributes associated with them.

## OpenID

OpenID is an open standard and decentralized authentication protocol. Promoted by the non-profit OpenID Foundation, it allows users to be authenticated by co-operating sites (known as relying parties, or RP) using a third-party service, eliminating the need for web masters to provide their own ad hoc login systems, and allowing users to log into multiple unrelated websites without having to have a separate identity and password for each.

## CAS

The Central Authentication Service (CAS) is a single sign-on protocol for the web. Its purpose is to permit a user to access multiple applications while providing their credentials (such as userid and password) only once. It also allows web applications to authenticate users without gaining access to a user's security credentials, such as a password.

## OAuth2.0

使用OAuth2来做认证是不安全的。因为authorization code是不包含可验证的用户信息，用户无法知道返回的这个code是不是来自合法的认证服务器。

HTTPS只能加密数据，无法防止CSRF。

> The fact that the communications between the browser and server is encrypted has no bearing on CSRF.

## JWT

JSON Web Token (JWT, sometimes pronounced /dʒɒt/) is an Internet standard for creating data with optional signature and/or optional encryption whose payload holds JSON that asserts some number of claims. The tokens are signed either using a private secret or a public/private key. For example, a server could generate a token that has the claim "logged in as admin" and provide that to a client. The client could then use that token to prove that it is logged in as admin. The tokens can be signed by one party's private key (usually the server's) so that party can subsequently verify the token is legitimate.

## OpenID Connect

OpenID Connect (OIDC) is an authentication layer on top of OAuth 2.0, an authorization framework.

1. The user attempts to start a session with your client app and is redirected to the OpenID Provider, passing in the client ID, which is unique for that application.
2. The OpenID Provider authenticates and authorizes the user for a particular application instance.
3. A one-time-use code is passed back to the web server using a predefined Redirect URI.
4. The web server passes the code, client ID, and client secret to the OpenID Provider’s token endpoint, and the OpenID Provider validates the code and returns a one-hour access token and JWT.
5. The web server uses the access token to get further details about the user and establishes a session for the user.

OpenID Connect allows a range of parties, including web-based, mobile and JavaScript clients, to request and receive information about authenticated sessions and end-users. The OpenID Connect specification is extensible, supporting optional features such as encryption of identity data, discovery of OpenID providers, and session management.