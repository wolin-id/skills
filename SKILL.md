---
name: wolin
description: Build Solid-OIDC apps and 3D worlds on Wolin Pods.
version: 0.1.0
author: Charles Choi (cycorld), Hermes Agent
license: MIT OR Apache-2.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [wolin, solid-oidc, dpop, metaverse, threejs, rdf, spatial-web]
    related_skills: []
---

# Wolin Sovereign Pod & Metaverse Skill

Integrate passwordless Wolin Passport authentication (W3C Solid-OIDC + RFC 9449 ES256 DPoP), cross-world 3D avatars (`https://wolin.id/ns#`), zero-egress live stages, and Zero-PII 3D spatial ads into web, mobile, Unity, and AI agent applications. Does not require API keys, OAuth client secrets, or manual app review.

## When to Use

- Adding "Continue with Wolin" passwordless sign-in to a web app, React Three Fiber world, Unity game, mobile app, or CLI/agent
- Reading or writing Solid LDP resources, containers, or `https://wolin.id/ns#` avatar/wearable triples on `{handle}.wolin.id`
- Embedding the 4 Wolin Metaverse Lego Blocks (`<WolinAuthAndAvatar />`, `<MediaStageScreen />`, `<WolinAdSlot />`, `public/client-id.jsonld`)
- Implementing MRC-grade 3D ad viewability (Frustum + >=5% screen area + Raycast occlusion + 1.5s continuous dwell)
- Don't use for: custodial wallet flows, PII-tracking ad pixels, or static API-key OAuth clients that break Solid-OIDC origin binding

## Prerequisites

- HTTPS deployment origin (or `http://localhost:<port>` during local development) serving a static JSON-LD Client ID Document at `<origin>/client-id.jsonld`
- Official SDKs by runtime:
  - TypeScript / React: `@wolin-id/client`, `@wolin-id/react`, or `wolin-id/metaverse-starter`
  - Python: `wolin-id` (`sdk-python`)
  - Swift / iOS / visionOS: `WolinSDK` (`sdk-swift`)
  - Kotlin / Android: `id.wolin:wolin-sdk` (`sdk-kotlin`)
  - Unity 3D: `id.wolin.sdk` (`sdk-unity`)
- Reference specifications in this skill:
  - `references/ns-schema.md` — `https://wolin.id/ns#` RDF vocabulary, Turtle examples, R2 private asset rules, and handle constraints
  - `references/lego-blocks.md` — 4 Metaverse Lego Blocks architecture, 3-tier media cost rules, and 3D ad viewability math

## Quick Reference

| Concept | Canonical Endpoint / IRI | Invariant |
| :--- | :--- | :--- |
| OIDC Issuer | `https://id.wolin.id` | Discovery at `/.well-known/openid-configuration` |
| WebID URI | `https://{handle}.wolin.id/profile/card#me` | Dereference with `Accept: text/turtle` |
| Pod Root | `https://{handle}.wolin.id/` | LDP Containers MUST end with `/` |
| Client ID | `https://<app-origin>/client-id.jsonld` | `origin(client_id) === origin(redirect_uri)` |
| 3D Avatar GLB | `https://wolin.id/ns#avatarGlbUrl` | Fallback to stylized 3D avatar if absent |
| 3D Avatar VRM | `https://wolin.id/ns#avatarVrmUrl` | Optional humanoid VRM 0.x/1.0 asset IRI |
| Passport Theme | `https://wolin.id/ns#passportTheme` | Accent token (e.g. `lunar-mercury`) |
| World Links | `https://wolin.id/ns#worldLinks` | Deep-link IRIs to user's active 3D worlds |
| Wearable Item | `https://wolin.id/ns#wearableItem` | Cross-world portable item identifier/IRI |

## Procedure

1. **Publish the Solid-OIDC Client ID Document.**
   Use `write_file` to create `public/client-id.jsonld` with `@context: ["https://www.w3.org/ns/solid/oidc-context.jsonld"]`, `dpop_bound_access_tokens: true`, `token_endpoint_auth_method: "none"`, and `redirect_uris` on the exact same scheme + host + port as `client_id`.
   *Completion criterion:* `new URL(doc.client_id).origin === new URL(doc.redirect_uris[0]).origin` evaluates to `true`.

2. **Execute PKCE `S256` + ES256 `DPoP` Authentication.**
   Discover endpoints from `<issuer>/.well-known/openid-configuration`, generate an RFC 7636 `S256` `code_verifier`/`code_challenge` and a non-extractable Web Crypto `ECDSA P-256` keypair, exchange the authorization code with a signed `dpop+jwt` header, and verify `<webId> solid:oidcIssuer <issuer>` in the dereferenced WebID Turtle document.
   *Completion criterion:* Token exchange succeeds (including automatic `DPoP-Nonce` `400`/`401` retry) and `verifyWebIdOidcIssuer` passes without error.

3. **Resolve `https://wolin.id/ns#` Avatar & Profile Metadata.**
   Read `references/ns-schema.md` with `read_file` and extract `wolin:avatarGlbUrl`, `wolin:avatarVrmUrl`, `wolin:passportTheme`, `wolin:worldLinks`, and `wolin:wearableItem` from the WebID profile. Always provide a deterministic 3D fallback avatar when `avatarGlbUrl` is unset or unreachable.
   *Completion criterion:* Both custom `.glb` profiles and empty/offline profiles render a valid 3D avatar with zero unhandled promise rejections.

4. **Assemble Media Stage & Spatial Ad Blocks (3D Worlds).**
   Read `references/lego-blocks.md` with `read_file`. Embed one-way broadcasts (YouTube Live / CHZZK Live) via 3D `<Html transform>` IFrames (`$0` server egress), use WebRTC P2P (`stun:stun.l.google.com:19302`) for 1:1–4p voice, and wire `<WolinAdSlot />` with Frustum + `>= 5%` screen area + `THREE.Raycaster` occlusion + `1500ms` continuous dwell checks.
   *Completion criterion:* Occluding the billboard with a mesh resets dwell time to `0ms`, and unobstructed `>= 1.5s` gaze emits a Zero-PII impression payload containing zero WebID or user-identifying fields.

5. **Enforce Easy UX Copy & Build Verification.**
   Audit UI labels with `search_files` to ensure end-user screens never display raw protocol terms (`DPoP`, `JWK`, `ETag`, `RDF`, `Turtle`). Run typecheck, unit tests, and production bundler via `terminal`.
   *Completion criterion:* `tsc --noEmit`, unit tests, and production build exit with code `0`.

## Pitfalls

- **Origin mismatch between `client_id` and `redirect_uri`:** Wolin Cloud rejects authorization requests unless `client_id` and `redirect_uri` share the exact same origin. Compute `${window.location.origin}/client-id.jsonld` dynamically in multi-environment apps.
- **Missing trailing slash on LDP Containers:** Requesting `https://alice.wolin.id/worlds` instead of `https://alice.wolin.id/worlds/` treats a container as a leaf resource and breaks relative IRI resolution.
- **Query or fragment in DPoP `htu`:** RFC 9449 §4.2 requires stripping `?query` and `#fragment` from `htu` before signing `dpop+jwt`; leaving `#me` in `htu` causes signature verification failure (`401`).
- **Skipping `DPoP-Nonce` retry:** Authorization and resource servers may return `400` or `401` with a `DPoP-Nonce` response header (`use_dpop_nonce`). Re-sign once with that `nonce` claim.
- **Counting occluded or oblique billboards as viewable:** Never credit a 3D ad impression from `Frustum` alone — always require front-facing normal (`cos >= 0.15`), `>= 5%` projected viewport area, unobstructed `Raycaster` line-of-sight, and `1.5s` continuous active dwell.
- **Z-index bleed from 3D `<Html>` overlays:** Wrap the WebGL `<Canvas>` in an isolated CSS stacking context (`isolation: isolate; z-index: 0`) so 3D CSS2D/CSS3D markers never punch through 2D modals (`z-index: 9999`).

## Verification

Run typechecking, unit tests, and production build through `terminal`:

```bash
npm run typecheck && npm test && npm run build
```

Confirm:
- PKCE `S256` + ES256 `DPoP` proof generation passes cryptographic verification.
- `parseWebIdProfile` extracts `https://wolin.id/ns#avatarGlbUrl` from Turtle documents.
- `evaluateSpatialAdFrame` returns `isViewable: false` when an occluder mesh blocks the raycast or screen area drops below `5%`.
