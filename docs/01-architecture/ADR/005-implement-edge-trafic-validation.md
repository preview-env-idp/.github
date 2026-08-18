# ADR 005: Implement JWT Cryptographic Validation for Edge Traffic Authentication

## Status

Accepted

## Context

External access to the cluster is securely routed through Cloudflare Tunnels (Zero Trust Network Access), with Cloudflare Access handling user authentication at the edge. However, by default, the internal Ingress Controller (Traefik) blindly trusts the `X-Forwarded-For` and identity headers forwarded by the `cloudflared` daemon.

If a malicious actor gains internal cluster access (e.g., via a compromised ephemeral tenant pod), they could bypass the edge authentication completely by sending direct HTTP requests to the Ingress Controller's internal IP, forging the required identity headers. This constitutes an Edge Authentication Bypass vulnerability.

## Decision

We will enforce cryptographic validation of incoming requests at the Ingress layer (Layer 7).

We will configure the Ingress Controller to intercept all traffic and cryptographically verify the JSON Web Token (JWT) provided by Cloudflare Access (in the `Cf-Access-Jwt-Assertion` header) against Cloudflare's public JSON Web Key Set (JWKS). Requests lacking a valid signature will be dropped with a `403 Forbidden` status.

## Consequences

* **Positive:** Ensures that all requests reaching the application have cryptographically proven their origin from the authenticated Cloudflare edge, entirely eliminating internal header-spoofing attacks.
* **Negative:** Adds a slight computational overhead to the Ingress Controller for RSA/ECDSA signature verification on every request.
* **Negative:** Requires initial configuration of Traefik middleware (e.g., ForwardAuth or JWT plugin) to continuously fetch and cache the public JWKS from Cloudflare.
