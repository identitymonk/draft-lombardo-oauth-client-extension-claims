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
    email: jeff@authnopuz.xyz
 -
    fullname: Alex Babeanu
    organization: IndieKite
    email: alex.babeanu@indykite.com

normative:
  RFC6749: # OAuth2
  RFC7519: # JSON Web Token
  RFC9068: # JWT profiled access Token
  RFC7591: # DCR
  RFC8693: # Token Exchange
  RFC8628: # Device Code
  OpenID.CIBA: #CIBA
  RFC8414: # OAuth 2.0 Authorization Server Metadata
  IANA.OAuth.Parameters: # IANA Registry

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
  RFC9421: # HTTP Message Signature
  RFC9700: # OAuth BCP
  I-D.ietf-winse-s2s-protocol: # WIMSE W2W
  I-D.ietf-oauth-transaction-tokens: # Transaction tokens


--- abstract

This specification defines new claims for JWT profiled access tokens {{RFC9068}} so that resource providers can benefit from more granular information about the client: its authentication methods as long as the grant flow and the grant flow extensions used as part of the issuance of the associated tokens.

--- middle

# Introduction

Resource providers need information about the subject, the action, the resource, and the context involved in the request to be able to determine properly, with the help of a Policy Decision Point (PDP) or not, if a resource must be disclosed.

When accessed with a JWT profiled OAuth2 Access Token {{RFC9068}} presented as a bearer token {{RFC6750}}, a resource provider receives mainly information about the subject in the form of:
- The `sub` claim,
- Any user profile claim set by the Authorization Server if applicable,
- Any Authenticaiton Information claims like the user class of authentication (`acr`claim) or user methord of authentication (claim `amr` {{RFC8176}})
- Any Authorization Information if applicable

The resource provider has very few information about the client, mainly in the form of the `client_id` {{RFC8693}} claim.

This document defines 4 new claims allowing to describe with more precise information the client and metadata on how it interacted with the authorization server during the issuance of the access token. It respects description of how to encode access tokens in JWT format.

The process by which the client interacts with the authorization server is out of scope.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

JWT access token:
    An OAuth 2.0 access token encoded in JWT format and complying with the requirements described in {{RFC9068}}.

This specification uses the terms "access token", "authorization server", "authorization endpoint", "authorization request", "client", "protected resource", and "resource server" defined by "The OAuth 2.0 Authorization Framework" {{RFC6749}}.

# JWT Access Token Client Extensions Data Structure

The following claims extend the {{RFC9068}} access token payload data structure:

## Client Flow Information Claims

The claims listed in this section MUST be issued and reflect grant type and extensions used with authorization server as part of the authorization request from the client. Their values are dynamic accros all access tokens that derive from a given authorization response to reflect the elements used in the process that lead to their issuance.

gty
    REQUIRED - defines the OAuth2 authorization grant type the client used for the issuance of the access token. String that is an identifier for an OAuth2 Grant type. Values used in the `gty` Claim MUST be from those registered in the IANA Grant Type Reference Values registry TODO established by this RFC and referencing, without being limited to, values established through section 2. of {{RFC7591}}, section 2.1 of {{RFC8693}}, and section 4 of {{OpenID.CIBA}}.

cxt
    REQUIRED - defines the list of extensions the client used in conjonction with the OAuth2 authorization grant type used for the issuance of the access token. For example but not limited to: Proof Key for Code Exchange by OAuth Public Clients (or PKCE) as defined in {{RFC 7636}}, Demonstrating Proof of Possession (or DPoP) as defined in {{RFC9449}}. JSON array of strings that are identifiers for extensions used. Values used in the `cxt` Claim MUST be from those registered in the IANA Client Context Reference Values registry TODO established by this RFC and referencing, without being limited to, values established through section 2 of {{RFC8414}}, and Section 5.1 of {{RFC9449}}.

## Client Authentication Information Claims

The claims listed in this section MAY be issued and reflect the types and strength of client authentication in the access token that the authentication server enforced prior to returning the authorization response to the client. Their values are fixed and remain the same across all access tokens that derive from a given authorization response, whether the access token was obtained directly in the response (e.g., via the implicit flow) or after obtaining a fresh access token using a refresh token. Those values may change if an access tokenis exchanged for another via an [RFC8693] procedure in order to reflect the specifities of this request.

ccr
    OPTIONAL - defines the authentication context class reference the client satisfied when authenticating to the authorization server. An absolute URI or registered name from furture RFC SHOULD be used as the `ccr` value; registered names MUST NOT be used with a different meaning than that which is registered. Parties using this claim will need to agree upon the meanings of the values used, which may be context specific.

cmr
    OPTIONAL - defines the authentication methods the client used when authenticating to the authorization server. String that is an identifier for an authentication method used in the authentication of the client. For instance, a value might indicate the usage of private JWT as defined in {{RFC7521}} and {{RFC7523}} or HTTP message signature as defined in {{RFC9421}} . The `cmr` value is a case-sensitive string. Values used in the `cmr` Claim SHOULD be from those registered in the IANA OAuth Token Endpoint Authentication Methods Values registry [IANA.OAuth.Parameters] defined by [RFC7591]; parties using this claim will need to agree upon the meanings of any unregistered values used, which may be context specific. 

# Authorization Server Metadata

The following authorization server metadata parameters [RFC8414] are introduced to signal the server's capability

support_client_extentison_claims
    Boolean parameter indicating whether the authorization server will return the extension claimns described in this RFC.

Note that the non presence of `support_client_extentison_claims` is sufficient for the client to determine that the server is not capable and therefore will not return the extension claimns described in this RFC. 

# Requesting a JWT Access Token with Client Extensions

An authorization server MUST issue a JWT access token with client extensions claims described in this RFC in response to any authorization grant defined by [RFC6749] and subsequent extensions meant to result in an access token and as along as the server support this capability.

# Validating JWT Access Tokens with Client Extensions

This specification follows the requirements of the section 4. of [RFC9068].

# Security Considerations

The JWT access token data layout described here is very similar to that of JWT access token as defined by [RFC9068].

The security current best practices described in [RFC9700] are applicable.

# IANA Considerations
## OAuth Grant Type Registration
This specification registers the following grant type in the [IANA.OAuth.Parameters] OAuth Grant Type registry.

### 'authorization_code' grant type

    Type name:
      authorization_code

   Change controller:
      IETF

   Specification document(s):
      section 2. of [RFC7591]

### 'implicit' grant type

    Type name:
      implicit

   Change controller:
      IETF

   Specification document(s):
      section 2. of [RFC7591]

### 'password' grant type

    Type name:
      password

   Change controller:
      IETF

   Specification document(s):
      section 2. of [RFC7591]

### 'client_credentials' grant type

    Type name:
      client_credentials

   Change controller:
      IETF

   Specification document(s):
      section 2. of [RFC7591]

### 'refresh_token' grant type

    Type name:
      refresh_token

   Change controller:
      IETF

   Specification document(s):
      section 2. of [RFC7591]

### 'urn:ietf:params:oauth:grant-type:jwt-bearer' grant type

    Type name:
      urn:ietf:params:oauth:grant-type:jwt-bearer

   Change controller:
      IETF

   Specification document(s):
      section 2. of {{RFC7591}}

### 'urn:ietf:params:oauth:grant-type:saml2-bearer' grant type

    Type name:
      urn:ietf:params:oauth:grant-type:saml2-bearer

   Change controller:
      IETF

   Specification document(s):
      section 2. of {{RFC7591}}

### 'urn:ietf:params:oauth:grant-type:token-exchange' grant type

    Type name:
      urn:ietf:params:oauth:grant-type:token-exchange

   Change controller:
      IETF

   Specification document(s):
      section 2.1. of {{RFC8693}}

### 'urn:ietf:params:oauth:grant-type:device_code' grant type

    Type name:
      urn:ietf:params:oauth:grant-type:device_code

   Change controller:
      IETF

   Specification document(s):
      section 3.4. of {{RFC8628}}

### 'urn:openid:params:grant-type:ciba' grant type

    Type name:
      urn:openid:params:grant-type:ciba

   Change controller:
      IETF

   Specification document(s):
      section 4. of {{OpenID.CIBA}}

## OAuth Grant Extension Type Registration
This specification registers the following grant extension type in the {{IANA.OAuth.Parameters}} OAuth Grant Extension Type registry.

### 'pkce' grant extension type

    Type name:
      pkce

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{RFC7636}}

### 'dpop' grant extension type

    Type name:
      dpop

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{RFC9449}}

### 'rar' grant extension type

    Type name:
      rar

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to  {{RFC9396}}

### 'par' grant extension type

    Type name:
      par

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{RFC9126}}

### 'jar' grant extension type

    Type name:
      jar

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{RFC9101}}

## OAuth Token Endpoint Authentication Methods Registration
This specification registers additional token endpoint authentication methods in the {{[IANA.OAuth.Parameters}} OAuth Token Endpoint Authentication Methods registry.

### 'jwt-bearer' authentication method type

    Type name:
      jwt-bearer

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{RFC7591}} and {{I-D.parecki-oauth-identity-assertion-authz-grant-02}}

### 'jwt_svid' authentication method type

    Type name:
      jwt_svid

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to [SPIFFE JWT-SVID](https://github.com/spiffe/spiffe/blob/main/standards/JWT-SVID.md)

### 'wit' authentication method type

    Type name:
      wit

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{I-D.ietf-wimse-s2s-protocol-03}}

### 'txn_token' authentication method type

    Type name:
      txn_token

   Change controller:
      IETF

   Specification document(s):
      This RFC as a reference to {{I-D.ietf-oauth-transaction-tokens-05}}

## Claims Registration
Section X.Y of this specification refers to the attributes "gty", "cxt", "ccr", and "cmr" to express client metadata JWT access tokens. This section registers those attributes as claims in the "JSON Web Token (JWT)" IANA registry introduced in {{RFC7519}}.

### Grant Type
Claim Name:
    gty
Claim Description:
    OAuth2 Grant Type used
Change Controller:
    IETF
Specification Document(s):
    Section X.Y of this document

### Client Extensions
Claim Name:
    cxt
Claim Description:
    Client Extensions used on top of the OAuth2 Grant Type
Change Controller:
    IETF
Specification Document(s):
    Section X.Y of this document

### Client Authentication Context
Claim Name:
    ccr
Claim Description:
    Client Authentication Class Reference
Change Controller:
    IETF
Specification Document(s):
    Section X.Y of this document

### Client Authentication Methods
Claim Name:
    cmr
Claim Description:
    Client Authentication Methods Reference
Change Controller:
    IETF
Specification Document(s):
    Section X.Y of this document

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
