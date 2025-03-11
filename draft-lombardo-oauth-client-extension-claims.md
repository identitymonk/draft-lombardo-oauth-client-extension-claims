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
  RFC6750: # OAuth2 Bearer Token Usage

informative:
  RFC8176: # AMR
  RFC7636: # PKCE
  RFC9396: # RAR
  RFC9126: # PAR
  RFC9101: # JAR
  RFC8693: # Token Exchange
  RFC9449: # DPOP
  RFC8705: # mTLS client


--- abstract

This specification defines new claims for JWT profiled access tokens {{RFC9068}} so that resource providers can benefit from more granular information about the client: its authentication methods as long as the grant flow and the grant flow extensions used as part of the issuance of the associated tokens.

--- middle

# Introduction

Resource providers need information about the subject, the action, the resource, and the context involved in the request to be able to determine properly, with the help of a Policy Decision Point (PDP) or not, if a resource must be disclosed.

When accessed with a JWT profiled access token {{RFC9068}} presented as a bearer token {{RFC6750}}, a resource provider receives mainly information about the subject:
- `sub` claim,
- Any user profile claim set by the Authorization Server if applicable,
- Any entitlement information if applicable,
- Any metadata about the user class of authentication (`acr`claim) or user methord of authentication (claim `amr` {{RFC8176}})

The resource provider has very few information about the client, mainly in the form of the `client_id` {{RFC8693}} claim.

This document defines 4 new claims allowing to describe more precise information about the client and metadata on how this client interacted with the authorization server during the issuance of the access token.

The process by which the client interacts with the authorization server is out of scope.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification uses the terms "access token", "authorization server", "authorization endpoint", "authorization request", "client", "protected resource", and "resource server" defined by "The OAuth 2.0 Authorization Framework" {{RFC6749}}.

# JWT Access Token Client Extensions Data Structure
## Client Authentication Information Claims
### Client Authentication Context claim
TODO
### Client Authentication Method claim
TODO
## Client Authorization Claims
### Grant Type claim
TODO
### Client Extensions claim
TODO


# Requesting a JWT Access Token with Client Extensions
TODO
# Validating JWT Access Tokens with Client Extensions
TODO
This specification follows the requirements of {{RFC9068}}.

# Security Considerations

TODO Security


# IANA Considerations
## Claims Registration
TODO of this specification refers to the attributes "gty", "cxt", "ccr", and "cmr" to express client metadata JWT access tokens. This section registers those attributes as claims in the "JSON Web Token (JWT)" IANA registry introduced in [RFC7519].

## Registry Content

### Grant Type
Claim Name:
    gty
Claim Description:
    Grant Type used
Change Controller:
    IETF
Specification Document(s):
    TODO of Draft

### Client Extensions
Claim Name:
    cxt
Claim Description:
    Client Extensions used for the Grant Type
Change Controller:
    IETF
Specification Document(s):
    TODO of Draft

### Client Authentication Context
Claim Name:
    ccr
Claim Description:
    Client Authentication Class Reference
Change Controller:
    IETF
Specification Document(s):
    TODO of Draft

### Client Authentication Methods
Claim Name:
    cmr
Claim Description:
    Client Authentication Methods Reference
Change Controller:
    IETF
Specification Document(s):
    TODO of Draft

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
