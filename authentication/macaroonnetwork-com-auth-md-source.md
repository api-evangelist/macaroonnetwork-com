# auth.md

Macaroon Network does not use OAuth registration, user accounts,
API keys, or bearer credentials for its public pay-per-call services.

## Authentication model

Public discovery endpoints require no identity.

Protected commercial resources use the x402 v2 payment protocol. Clients
receive HTTP 402 Payment Required
with payment requirements (`PAYMENT-REQUIRED`) and authorize payment
according to the advertised scheme, then retry with `PAYMENT-SIGNATURE`
and receive `PAYMENT-RESPONSE`.

Successful payment verification authorizes only the requested resource
transaction. It does not create an account, OAuth session, bearer
token, API key, or persistent identity.

## Identity

identity_types_supported:
- anonymous

No personally identifying registration is required for the public
x402 payment flow. An agent may optionally send a self-assigned
`X-Macaroon-Agent-Id` header purely to scope its own free-tier quota --
this is never an identity or trust credential.

## OAuth

OAuth is not implemented by Macaroon Network.

The RFC 9728 protected-resource metadata document at
[/.well-known/oauth-protected-resource](https://api.macaroonnetwork.com/.well-known/oauth-protected-resource)
is published for machine-readable resource discovery, but it
intentionally advertises no authorization server.

## Payment discovery

Service descriptions:
https://api.macaroonnetwork.com/.well-known/api-catalog

API:
https://api.macaroonnetwork.com

x402 version:
v2

## Credential use

The payment proof is not a bearer credential for anything beyond the single
call it was issued for --
there is nothing to store, refresh, or revoke. Discover current
listings, prices, and payment rails at
[/.well-known/ai-catalog.json](https://api.macaroonnetwork.com/.well-known/ai-catalog.json).
