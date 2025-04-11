---
title: "OAuth 2.0 client extension claims"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-lombardo-oauth-client-extension-claims-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Web Authorization Protocol"
keyword:
 - security
 - trust
venue:
  group: "Web Authorization Protocol"
  type: ""
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "identitymonk/draft-lombardo-oauth-client-extension-claims"
  latest: "https://identitymonk.github.io/draft-lombardo-oauth-client-extension-claims/draft-lombardo-oauth-client-extension-claims.html"

author:
 -
    fullname: Jeff Lombardo
    organization: AWS
    country: Canada
    email: jeff@authnopuz.xyz

 -
    fullname: Alexandre Babeanu
    organization: IndyKite
    country: Canada
    email: alex.babeanu@indykite.com

normative:
  RFC6749: # OAuth2
  RFC7519: # JSON Web Token
  RFC9068: # JWT profiled access Token
  RFC7591: # DCR
  RFC8693: # Token Exchange
  RFC8628: # Device Code
  OpenID.CIBA:
    target: https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html
    title: OpenID Connect Client-Initiated Backchannel Authentication Flow - Core 1.0
    date: 1 September 2021
    author:
      -
        name: G. Fernandez
      -
        name: F. Walter
      -
        name: A. Nennker
      -
        name: D. Tonge
      -
        name: B. Campbell
  RFC8414: # OAuth 2.0 Authorization Server Metadata
  IANA.oauth-parameters: # IANA Registry

informative:
  RFC6750: # OAuth2 Bearer Token Usage
  RFC7521: # Private JWT
  RFC7523: # Private JWT
  RFC8176: # AMR
  RFC7636: # PKCE
  RFC9396: # RAR
  RFC9126: # PAR
  RFC9101: # JAR
  RFC9449: # DPOP
  RFC8705: # mTLS client
  RFC9421: # HTTP Message Signaturenano /etc/update-manager/release-upgrades
  RFC9700: # OAuth BCP
  I-D.parecki-oauth-identity-assertion-authz-grant: # Assertion grant
  I-D.ietf-wimse-s2s-protocol: # WIMSE W2W
  I-D.ietf-oauth-transaction-tokens: # Transaction tokens
  IANA.oauth-parameters_token-endpoint-auth-method: # IANA Registry
  IANA.jwt: # IANA registry


--- abstract

This specification defines new claims for JWT profiled access tokens {{RFC9068}} so that resource providers can benefit from more granular information about the client to make better informed access decisions. The proposed new claims include: the client authentication methods, the client OAuth grant flow used as well as the OAuth grant flow extensions used as part of the issuance of the associated tokens.

--- middle

# Introduction

Resource providers need information about the subject, the action, the resource, and the context involved in the request in order to be able to determine properly if a resource can be disclosed. This decision may also involve the help of a Policy Decision Point (PDP).

When accessed with a JWT profiled OAuth2 Access Token {{RFC9068}} presented as a bearer token {{RFC6750}}, a Resource Server (RS) receives information about the subject as claims, such as:
- The subject,  `sub`,
- Any user profile claim set by the Authorization Server if applicable,
- Authenticaton Information claims like the user class of authentication (`acr`claim) or user methord of authentication (claim `amr` {{RFC8176}})
- Any Authorization Information if applicable

On the other hand, the RS has very little information about the client, often only a `client_id` {{RFC8693}} claim. In particular, the RS lacks any insight about the type of authorization grant flow that the client itself went through, the level of assurance that the client itself may have reached during its own authentication, and in general, lacks any information that would enable it to determine if the client itself can be trusted at all.

This document defines 4 new claims for the JWT profile of OAuth access tokens, which describe in detail the interaction between the OAuth client and the Authorization Server (AS) during the flow that resulted in the issuance of the token.

The process by which the client interacts with the authorization server is out of scope.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

_JWT access token_:
: An OAuth 2.0 access token encoded in JWT format and complying with the requirements described in {{RFC9068}}.

This specification uses the terms "access token", "authorization server", "authorization endpoint", "authorization request", "client", "protected resource", and "resource server" defined by "The OAuth 2.0 Authorization Framework" {{RFC6749}}.

# JWT Access Token Client Extensions Data Structure

The following claims extend the {{RFC9068}} access token payload data structure:

## Client Flow Information Claims

The claims listed below reflect the grant type and extensions used by the OAuth client with the Authorization Server during the authorization request. Their values are generated dynamically and consistent across all the tokens generated for the given authorization request.

`gty`:
: _REQUIRED_ - a string that describes the OAuth2 authorization grant type the client requested for the issuance of the access token. The value used as the `gty` claim MUST comply to existing grant flows described in the following specifications:  section 1.3 of {{RFC6749}}, section 2. of {{RFC7591}}, section 2.1 of {{RFC8693}}, and section 4 of {{OpenID.CIBA}}. Furthermore since future grants may also be developped, this claim is not limited to existing flows, but should also encompass future innovations. The possible values of this claim are registered with IANA (see below). Non-normative example value: "`client_credentials`".

`cxt`:
: _REQUIRED_ - defines the list of extensions the client used in conjonction with the OAuth2 authorization grant type used for the issuance of the access token. For example but not limited to: Proof Key for Code Exchange by OAuth Public Clients (or PKCE) as defined in {{RFC7636}}, Demonstrating Proof of Possession (or DPoP) as defined in {{RFC9449}}. The claim value is an array of strings that lists the identifiers of extensions used. Values used in the `cxt` Claim MUST be valid values registered with the IANA OAuth Parameters registry @TODO. These values reference, without being limited to, values established through section 2 of {{RFC8414}}, and Section 5.1 of {{RFC9449}}. Non-normative example: "`[dpop,pkce]`".

## Client Authentication Information Claims

The claims listed in this section MAY be issued and reflect strength of the mechanism used to authenticate the OAuth2 Client client itself as part of the  access token issuance flow. Their values are fixed and remain the same across all access tokens that derive from a given authorization response, whether the access token was obtained directly in the response (e.g., via the implicit flow) or after obtaining a new access token using a refresh token. Those values may change if an access token is exchanged for another through the Token Exchange {{RFC8693}} procedure, in that case these claims will reflect the details of this new request.

`ccr`:
: _OPTIONAL_ - refers to the authentication context class that the Oauth client achieved with the AS during the authorization flow. The value of this claim must be an absolute URI that can be registered with IANA. It should support present, future or custom values. If IANA registered URIs are used, then their meaning and semantics should be respected and used as defined in the registry. Parties using this custom claim values need to agree upon the semantics of the values used, which may be context specific. Non-normative example: "`urn:org:iana:client:assurance:level_1`".

`cmr`:
: _OPTIONAL_ - an Identifier String that defines the authentication methods the client used when authenticating to the authorization server. The claim may indicate the usage of private JWT as defined in {{RFC7521}} and {{RFC7523}} or HTTP message signature as defined in {{RFC9421}}, or simple client secret as defined in {{RFC6749}} . The `cmr` value is a case-sensitive string, which values SHOULD be registered with the IANA OAuth Token Endpoint Authentication Methods Values registry {{IANA.oauth-parameters_token-endpoint-auth-method}} defined by {{RFC7591}}; parties using this claim will need to agree upon the meanings of any unregistered values used, which may be context specific.

# Authorization Server Metadata

Not all Authorizartion Servers (AS) may support the claims described in this document. It is therefore necessary to provide a way for an OAuth Resource Server to determine whether it can rely on the claims described here. This document therefore proposes the following extension to the OAuth2.0 Authorization Server Metadata {{RFC8414}} specification; i.e., the following metadata value is to be considered an extension to the values described in section 2 of the OAuth2.0 Authorization Server Metadata {{RFC8414}} specification:

`support_client_extentison_claims`:
: Boolean parameter indicating whether the authorization server will return the extension claims described in this RFC.

Note that the non presence of `support_client_extentison_claims` is sufficient for the client to determine that the server is not capable and therefore will not return the extension claimns described in this RFC. This ensures backward compatibility with all existing AS implementations.

# Requesting a JWT Access Token with Client Extensions

An authorization server supporting the claims described in this document MUST issue a JWT access token with client extensions claims described in this RFC in response to any authorization grant defined by [RFC6749], as well as subsequent extensions meant to result in an access token.

# Validating JWT Access Tokens with Client Extensions

This specification does not change any of the requirements for validating access tokens, as defined in section 4 of the JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens specification {{RFC9068}}.

# Security Considerations

The JWT access token data layout described here is the same as the struture of the JWT access token as defined by the JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens specification {{RFC9068}}.

The Best Current Practice for OAuth 2.0 Security {{RFC9700}} is still applicable.

# IANA Considerations

## OAuth Grant Type Registration

This specification registers the following grant type in the {{IANA.oauth-parameters}} OAuth Grant Type registry.

### `authorization_code` grant type

   Type name:
   : `authorization_code`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `implicit` grant type

   Type name:
   : `implicit`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `password` grant type

   Type name:
   : `password`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `client_credentials` grant type

   Type name:
   : `client_credentials`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `refresh_token` grant type

   Type name:
   : `refresh_token`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `jwt-bearer` grant type

   Type name:
   : `urn:ietf:params:oauth:grant-type:jwt-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `saml2-bearer` grant type

   Type name:
   : `urn:ietf:params:oauth:grant-type:saml2-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of {{RFC7591}}

### `token-exchange` grant type

   Type name:
   : `urn:ietf:params:oauth:grant-type:token-exchange`

   Change controller:
   : IETF

   Specification document(s):
   : section 2.1. of {{RFC8693}}

### `device_code` grant type

   Type name:
   : `urn:ietf:params:oauth:grant-type:device_code`

   Change controller:
   : IETF

   Specification document(s):
   : section 3.4. of {{RFC8628}}

### `ciba` grant type

   Type name:
   : `urn:openid:params:grant-type:ciba`

   Change controller:
   : IETF

   Specification document(s):
   : section 4. of {{OpenID.CIBA}}

## OAuth Grant Extension Type Registration

This specification registers the following grant extension type in the {{IANA.oauth-parameters}} OAuth Grant Extension Type registry.

### `pkce` grant extension type

   Type name:
   : `pkce`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{RFC7636}}

### `dpop` grant extension type

   Type name:
   : `dpop`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{RFC9449}}

### `rar` grant extension type

   Type name:
   : `rar`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to  {{RFC9396}}

### `par` grant extension type

   Type name:
   : `par`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{RFC9126}}

### `jar` grant extension type

   Type name:
   : `jar`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{RFC9101}}

## OAuth Token Endpoint Authentication Methods Registration

This specification registers additional token endpoint authentication methods in the {{IANA.oauth-parameters}} OAuth Token Endpoint Authentication Methods registry.

### `jwt-bearer` token endpoint authentication method

   Type name:
   : `jwt-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{RFC7591}} and {{I-D.parecki-oauth-identity-assertion-authz-grant}}

### `jwt-svid` token endpoint authentication method

   Type name:
   : `jwt-svid`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [SPIFFE JWT-SVID](https://github.com/spiffe/spiffe/blob/main/standards/JWT-SVID.md)

### `wit` token endpoint authentication method

   Type name:
   : `wit`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{I-D.ietf-wimse-s2s-protocol}}

### `txn_token` token endpoint authentication method

   Type name:
   : `txn_token`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to {{I-D.ietf-oauth-transaction-tokens}}

## Claims Registration

Section X.Y of this specification refers to the attributes `gty`, `cxt`, `ccr`, and `cmr` to express client metadata JWT access tokens. This section registers those attributes as claims in the {{IANA.jwt}} registry introduced in {{RFC7519}}.

### `gty` claim definition

   Claim Name:
   : `gty`

   Claim Description:
   : OAuth2 Grant Type used

   Change Controller:
   : IETF

   Specification Document(s):
      Section X.Y of this document

### `cxt` claim definition

   Claim Name:
   : `cxt`

   Claim Description:
   : Client Extensions used on top of the OAuth2 Grant Type

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

### `ccr` claim definition

   Claim Name:
   : `ccr`

   Claim Description:
   : Client Authentication Class Reference

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

### `cmr` claim definition

   Claim Name:
   : `cmr`

   Claim Description:
   : Client Authentication Methods Reference

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
