---
title: "OAuth 2.0 client extension claims"
abbrev: "OCEC"
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
    fullname: Jean-François Lombardo
    nickname: Jeff
    organization: AWS
    country: Canada
    email: jeffsec@amazon.com

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
        name: Gonzalo Fernandez Rodriguez
        role: editor
        org: Telefonica
      -
        name: Florian Walter
        role: editor
        org: Deutsche Telekom AG
      -
        name: Axel Nennker
        role: editor
        org: Deutsche Telekom AG
      -
        name: Dave Tonge
        role: editor
        org: Moneyhub
      -
        name: Brian Campbell
        role: editor
        org: Ping Identity
  RFC8414: # OAuth 2.0 Authorization Server Metadata
  IANA.oauth-parameters: # IANA Registry

informative:
  RFC6750: # OAuth2 Bearer Token Usage
  RFC7521: # Private JWT
  RFC7523: # JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants
  RFC8176: # AMR
  RFC7636: # PKCE
  RFC9396: # RAR
  RFC9126: # PAR
  RFC9101: # JAR
  RFC9449: # DPOP
  RFC8705: # mTLS client
  RFC9421: # HTTP Message Signature
  RFC9700: # OAuth BCP
  I-D.parecki-oauth-identity-assertion-authz-grant: # Assertion grant
  I-D.ietf-wimse-s2s-protocol: # WIMSE W2W
  I-D.ietf-oauth-transaction-tokens: # Transaction tokens
  IANA.oauth-parameters_token-endpoint-auth-method: # IANA Registry
  IANA.jwt: # IANA registry
  FAPI2.0-Security-Profiles:
    title: FAPI 2.0 Security Profile
    target: https://openid.net/specs/fapi-2_0-security-02.html
    author:
    - name: Dr Daniel Fett
      role: editor
      org: Authlete
    - name: Dave Tonge
      role: editor
      org: Moneyhub Financial Technology
    - name: Joseph Heenan
      role: editor
      org: Authlete
  hl7.fhir.uv.smart-app-launch:
    title: HL7 FHIR SMART App Launch
    target: https://www.hl7.org/fhir/smart-app-launch/app-launch.html#obtain-authorization-code

--- abstract

This specification defines new claims for JWT profiled access tokens [RFC9068] so that resource providers can benefit from more granular information about the client: its authentication methods as well as the grant flow and the grant flow extensions used as part of the issuance of the associated tokens.

--- middle

# Introduction {#introduction}

Resource providers need information about the subject, the action, the resource, and the context involved in the request in order to be able to determine properly if a resource can be disclosed. This decision may also involve the help of a Policy Decision Point (PDP).

When accessed with a JWT profiled OAuth2 Access Token [RFC9068] presented as a bearer token [RFC6750], a resource provider receives mainly information about the subject in the form of:
- The `sub` claim,
- Any user profile claim set by the Authorization Server if applicable,
- Any Authentication Information claims like the user class of authentication (`acr`claim) or user method of authentication (claim `amr` [RFC8176])
- Any Authorization Information if applicable

The resource provider has very little information about the client, mainly in the form of the `client_id` [RFC8693] claim. It falls short in several important circumstances, for instance, in [FAPI2.0-Security-Profiles] or [hl7.fhir.uv.smart-app-launch] regulated APIs when they require peculiar client authentication mechanisms to be enforced or transaction specific details to be present in the token.

This document defines 4 new claims allowing to describe with more precise information the client and metadata on how it interacted with the authorization server during the issuance of the access token. It respects description of how to encode access tokens in JWT format.

The process by which the client interacts with the authorization server is out of scope.

# Conventions and Definitions {#conventions-definitions}

{::boilerplate bcp14-tagged}

JWT access token:
: An OAuth 2.0 access token encoded in JWT format and complying to the requirements described in [RFC9068].

This specification uses the terms "access token", "authorization server", "authorization endpoint", "authorization request", "client", "protected resource", and "resource server" defined by "The OAuth 2.0 Authorization Framework" [RFC6749].

# JWT Access Token Client Extensions Data Structure {#client-ext-data-structure}

The following claims extend the [RFC9068] access token payload data structure:

## Client Flow Information Claims {#client-flow-info}

The claims listed in this section MUST be issued and reflect grant type and extensions used with authorization server as part of the authorization request from the client. Their values are dynamic across all access tokens that derive from a given authorization response to reflect the elements used in the process that lead to their issuance.

`gty`:
: REQUIRED - defines the OAuth2 authorization grant type the client used for the issuance of the access token. String that is an identifier for an OAuth2 Grant type. Values used in the `gty` Claim MUST be from those registered in the IANA Grant Type Reference Values registry TODO established by this RFC and referencing, without being limited to, values established through section 2. of [RFC7591], section 2.1 of [RFC8693], and section 4 of [OpenID.CIBA].

`cxt`:
: REQUIRED - defines the list of extensions the client used in conjunction with the OAuth2 authorization grant type used for the issuance of the access token. For example but not limited to: Proof Key for Code Exchange by OAuth Public Clients (or PKCE) as defined in [RFC7636], Demonstrating Proof of Possession (or DPoP) as defined in [RFC9449]. JSON array of strings that are identifiers for extensions used. Values used in the `cxt` Claim MUST be from those registered in the IANA Client Context Reference Values registry TODO established by this RFC and referencing, without being limited to, values established through section 2 of [RFC8414], and Section 5.1 of [RFC9449].

## Client Authentication Information Claims {#client-authn-info}

The claims listed in this section MAY be issued and reflect the types and strength of client authentication in the access token that the authentication server enforced prior to returning the authorization response to the client. Their values are fixed and remain the same across all access tokens that derive from a given authorization response, whether the access token was obtained directly in the response (e.g., via the implicit flow) or after obtaining a fresh access token using a refresh token. Those values may change if an access token is exchanged for another via an [RFC8693] procedure in order to reflect the specificities of this request.

`ccr`:
: OPTIONAL - defines the authentication context class reference the client satisfied when authenticating to the authorization server. An absolute URI or registered name from future RFC SHOULD be used as the `ccr` value; registered names MUST NOT be used with a different meaning than that which is registered. Parties using this claim will need to agree upon the meanings of the values used, which may be context specific.

`cmr`:
: OPTIONAL - defines the authentication methods the client used when authenticating to the authorization server. String that is an identifier for an authentication method used in the authentication of the client. For instance, a value might indicate the usage of private JWT as defined in [RFC7521] and [RFC7523] or HTTP message signature as defined in [RFC9421] . The `cmr` value is a case-sensitive string. Values used in the `cmr` Claim SHOULD be from those registered in the IANA OAuth Token Endpoint Authentication Methods Values registry [IANA.oauth-parameters_token-endpoint-auth-method] defined by [RFC7591]; parties using this claim will need to agree upon the meanings of any unregistered values used, which may be context specific.

# Authorization Server Metadata {#as-metadata}

The following authorization server metadata parameters are introduced as an extension of [RFC8414], in order to describe the server's capabilities.

`support_client_extentison_claims`:
: Boolean parameter indicating to clients and resource servers whether the authorization server will return the extension claims described in this document.

Note: that the non presence of `support_client_extentison_claims` is sufficient for the client to determine that the server is not capable and therefore will not return the extension claims described in this RFC.

# Requesting a JWT Access Token with Client Extensions {#request-jwt-client-ext}

This specification does not change how clients interacts with authorization servers.

An authorization server supporting this specification MUST issue a JWT access token with client extensions claims described in this RFC in response to any authorization grant defined by [RFC6749] and subsequent extensions meant to result in an access token and as along as the authorization server support this capability.

# Validating JWT Access Tokens with Client Extensions {#validate-jwt-client-ext}

This specification follows the requirements of the section 4 of [RFC9068].

# Security Considerations {#security}

## Validation Of Token

The JWT access token data format described here is the same as JWT access token defined by [RFC9068].

## Processing Of Claims Defined By This Document

Any processor, client or resource server, MUST only process claims described in this document that it understands.

If a processor does not understand a claim described in this document or its value, it SHOULD ignore it.

## Security Best Practice

The security current best practices described in [RFC9700] MUST be applied.

# IANA Considerations {#iana}

The following registration procedure is used for the registry established by this specification.

Values are registered on a Specification Required [RFC8126] basis after a two-week review period on the oauth-ext-review@ietf.org mailing list, on the advice of one or more Designated Experts. However, to allow for the allocation of values prior to publication of the final version of a specification, the Designated Experts may approve registration once they are satisfied that the specification will be completed and published. However, if the specification is not completed and published in a timely manner, as determined by the Designated Experts, the Designated Experts may request that IANA withdraw the registration.

Registration requests sent to the mailing list for review should use an appropriate subject (e.g., "Request to register JWT profiled OAuth2 Access Token client extensions: example").

Within the review period, the Designated Experts will either approve or deny the registration request, communicating this decision to the review list and IANA. Denials should include an explanation and, if applicable, suggestions as to how to make the request successful. The IANA escalation process is followed when the Designated Experts are not responsive within 14 days.

Criteria that should be applied by the Designated Experts includes determining whether the proposed registration duplicates existing functionality, determining whether it is likely to be of general applicability or whether it is useful only for a single application, and whether the registration makes sense.

IANA must only accept registry updates from the Designated Experts and should direct all requests for registration to the review mailing list.

It is suggested that multiple Designated Experts be appointed who are able to represent the perspectives of different applications using this specification, in order to enable broadly-informed review of registration decisions. In cases where a registration decision could be perceived as creating a conflict of interest for a particular Expert, that Expert should defer to the judgment of the other Experts.

The reason for the use of the mailing list is to enable public review of registration requests, enabling both Designated Experts and other interested parties to provide feedback on proposed registrations. The reason to allow the Designated Experts to allocate values prior to publication as a final specification is to enable giving authors of specifications proposing registrations the benefit of review by the Designated Experts before the specification is completely done, so that if problems are identified, the authors can iterate and fix them before publication of the final specification.

## OAuth Grant Type Registration {#iana-grant-type-reg}

This specification registers the following grant type in the [IANA.oauth-parameters] OAuth Grant Type registry.

### `authorization_code` grant type {#iana-grant-type-reg-ac}

   Type name:
   : `authorization_code`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `implicit` grant type {#iana-grant-type-implicit}

   Type name:
   : `implicit`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `password` grant type {#iana-grant-type-reg-pwd}

   Type name:
   : `password`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `client_credentials` grant type {#iana-grant-type-reg-cc}

   Type name:
   : `client_credentials`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `refresh_token` grant type {#iana-grant-type-rt}

   Type name:
   : `refresh_token`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `jwt-bearer` grant type {#iana-grant-type-jwt}

   Type name:
   : `urn:ietf:params:oauth:grant-type:jwt-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591] and section 6 of [I-D.parecki-oauth-identity-assertion-authz-grant]

### `saml2-bearer` grant type {#iana-grant-type-reg-saml2}

   Type name:
   : `urn:ietf:params:oauth:grant-type:saml2-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : section 2. of [RFC7591]

### `token-exchange` grant type {#iana-grant-type-reg-te}

   Type name:
   : `urn:ietf:params:oauth:grant-type:token-exchange`

   Change controller:
   : IETF

   Specification document(s):
   : section 2.1. of [RFC8693]

### `device_code` grant type {#iana-grant-type-reg-dc}

   Type name:
   : `urn:ietf:params:oauth:grant-type:device_code`

   Change controller:
   : IETF

   Specification document(s):
   : section 3.4. of [RFC8628]

### `ciba` grant type {#iana-grant-type-reg-ciba}

   Type name:
   : `urn:openid:params:grant-type:ciba`

   Change controller:
   : IETF

   Specification document(s):
   : section 4. of [OpenID.CIBA]

## OAuth Grant Extension Type Registration {#iana-grant-ext-reg}

This specification registers the following grant extension type in the [IANA.oauth-parameters] OAuth Grant Extension Type registry.

### `pkce` grant extension type {#iana-grant-ext-reg-pkce}

   Type name:
   : `pkce`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC7636]

### `dpop` grant extension type {#iana-grant-ext-reg-dpop}

   Type name:
   : `dpop`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC9449]

### `wpt` grant extension type {#iana-grant-ext-reg-wpt}

   Type name:
   : `wpt`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to section 4.2 of [I-D.ietf-wimse-s2s-protocol]

### `rar` grant extension type {#iana-grant-ext-rar}

   Type name:
   : `rar`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC9396]

### `par` grant extension type {#iana-grant-ext-reg-par}

   Type name:
   : `par`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC9126]

### `jar` grant extension type {#iana-grant-ext-reg-jar}

   Type name:
   : `jar`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC9101]

## OAuth Token Endpoint Authentication Methods Registration {#iana-token-authn-reg}

This specification registers additional token endpoint authentication methods in the [IANA.oauth-parameters] OAuth Token Endpoint Authentication Methods registry.

### `jwt-bearer` token endpoint authentication method {#iana-token-authn-reg-jwt}

   Type name:
   : `jwt-bearer`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [RFC7591] and [I-D.parecki-oauth-identity-assertion-authz-grant]

### `jwt-svid` token endpoint authentication method {#iana-token-authn-reg-jwt-svid}

   Type name:
   : `jwt-svid`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [SPIFFE JWT-SVID](https://github.com/spiffe/spiffe/blob/main/standards/JWT-SVID.md)

### `wit` token endpoint authentication method {#iana-token-authn-reg-wit}

   Type name:
   : `wit`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [I-D.ietf-wimse-s2s-protocol]

### `txn_token` token endpoint authentication method {#iana-token-authn-reg-txntoken}

   Type name:
   : `txn_token`

   Change controller:
   : IETF

   Specification document(s):
   : This RFC as a reference to [I-D.ietf-oauth-transaction-tokens]

## Claims Registration {#iana-claims-reg}

Section X.Y of this specification refers to the attributes `gty`, `cxt`, `ccr`, and `cmr` to express client metadata JWT access tokens. This section registers those attributes as claims in the [IANA.jwt] registry introduced in [RFC7519].

### `gty` claim definition {#iana-claims-reg-gty}

   Claim Name:
   : `gty`

   Claim Description:
   : OAuth2 Grant Type used

   Change Controller:
   : IETF

   Specification Document(s):
      Section X.Y of this document

### `cxt` claim definition {#iana-claims-cxt}

   Claim Name:
   : `cxt`

   Claim Description:
   : Client Extensions used on top of the OAuth2 Grant Type

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

### `ccr` claim definition {#iana-claims-ccr}

   Claim Name:
   : `ccr`

   Claim Description:
   : Client Authentication Class Reference

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

### `cmr` claim definition {#iana-claims-cmr}

   Claim Name:
   : `cmr`

   Claim Description:
   : Client Authentication Methods Reference

   Change Controller:
   : IETF

   Specification Document(s):
   : Section X.Y of this document

--- back

# Acknowledgments {#ack}
{:numbered="false"}


The authors wants to acknowledge the support and work of the following individuals: George Fletcher (Practical Identity), Christopher Langton (Vulnetix).

The authors wants also to recognize the trail blazers and thought leaders that created the ecosystem without which this draft proposal would not be able to solve customer pain points and secure usage of digital services, especially without being limited to: Vittorio Bertocci†, Brian Campbell (Ping Identity), Justin Richer (MongoDB), Aaron Parecki (Okta), Pieter Kasselman (SPRL), Dr Mike Jones (Self-Issued Consulting, LLC), Dr Daniel Fett (Authlete).
