# Good knowledge of security (SAML, OAuth, OpenID, Kerberos, Policies, entitlements etc.)

SAML (Security Assertion Markup Language), OAuth (Open Authorization), OIDC (OpenID Connect), and Kerberos are widely used **SSO protocols** for authentication and authorization. While they share the goal of securing access to systems and resources, each serves distinct purposes and is suited for specific scenarios.

## Authentication vs Authorization (foundation)

- Authentication: Authentication is the process of verifying who a user is. 

- Authorization: Authorization determines the user's level of access.

### Explanation

You always authenticate first, then authorization decides permissions based on roles, policies, or entitlements.

```pgsql
User ──(login)──> Identity Check ──> Access Decision
        AuthN              AuthZ
```

## Purpose

- SAML: Authentication

- OAuth: Authorization

- OIDC: Authentication + Authorization

- Kerberos: Authentication in trusted environments

## Protocol Type / Data Format

- SAML: XML

- OAuth: JSON

- OIDC: JSON

- Kerberos: Ticket-based, binary data format

### When to Use

#### SAML:

- Best suited for legacy systems.
- This does not apply to APIs
- Not mobile-friendly

#### OAuth:

- Not intended for Web-based SSOs.
- Perfect for API access.
- Excellent for mobile and cloud apps.

#### OIDC:

- Ideal for modern Web-based SSO.
- Best suited for mobile and cloud apps.

#### Kerberos:

Best for internal enterprise environments, internal application.
Not applicable for APIs and Mobile.

# Security Concepts

## SAML (Security Assertion Markup Language)


- SAML is an XML-based protocol used for enterprise Single Sign-On (SSO) between an Identity Provider and an application.

### Explanation

The application trusts the Identity Provider. Once the user is authenticated, the IdP sends a signed/SAML assertion proving identity to the application and application grants access based on the assertion.

```pgsql
User
  |
  | 1. Access App
  v
Service Provider (App)
  |
  | 2. Redirect
  v
Identity Provider (Okta / ADFS)
  |
  | 3. Authenticate
  v
SAML Assertion
  |
  | 4. Trust & Login
  v
Application Access
```

## OAuth 2.0

- OAuth 2.0 is an authorization framework that allows applications to access resources on behalf of a user or service.

### Explanation

- OAuth does not prove identity. It issues access tokens that grant limited permissions to APIs.

- User authorizes a third-party app (client) to access resources. The client (third-party app) receives an authorization code from the Authorization Server. The client (third-party app) exchanges the code for an access token. The access token is used to access the resource server.

```pgsql
User
  |
  | Approves access
  v
Authorization Server
  |
  | Access Token
  v
Client App ──> Protected API
```

## OpenID Connect (OIDC)

- OpenID Connect is an authentication layer built on top of OAuth 2.0.

### Explanation

OIDC adds an ID token alongside an access token that tells the application who the user is: The ID Token contains identity details like user name and email (Authentication). while OAuth handles what they can access (Authorization).

```pgsql
User
  |
  v
OIDC Provider
  |        \
  |         \
ID Token   Access Token
  |             |
  v             v
Identity     API Access
```

## Kerberos

- Kerberos is a ticket-based authentication protocol commonly used in enterprise and Active Directory environments.

### Explanation

The user authenticates with the **KDC** once and receives a **TGT**. The **TGT** is used to request service tickets for specific applications. The service ticket allows secure access to the requested service.

```pgsql
User
  |
  | Login
  v
Key Distribution Center (KDC)
  |
  | Ticket Granting Ticket (TGT)
  v
Service Ticket ──> Protected Service
```

## Policies

- Policies define rules that allow or deny actions on resources.

### Explanation

Policies are evaluated during authorization to decide whether a request is permitted.

```pgsql
Request
  |
  v
Policy Engine
  |
  | Allow / Deny
  v
Resource Access
```

## Entitlements

- Entitlements are the effective permissions a user or service actually has.

### Explanation

They result from combining roles, group membership, and policies.

```pgsql
User
  |
  v
Roles + Policies
  |
  v
Effective Entitlements
```

## How Interviewers Expect You to Connect This

Authentication proves identity, authorization enforces access using policies and entitlements, and protocols like SAML, OAuth, OIDC, and Kerberos implement these concepts across enterprise, cloud, and DevOps systems.