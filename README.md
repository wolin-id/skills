# Wolin (月印千江) Agent Skills (`wolin-id/skills`)

> Official AI Coding Agent Skills & Protocol References for **Wolin (月印千江 · `https://wolin.id`)** — Sovereign Solid 0.11 Pods, Passwordless Solid-OIDC + DPoP Identity, `https://wolin.id/ns#` 3D Avatar Interop, and Zero-PII Spatial AdSense.

Designed for **Claude Code**, **Cursor**, **Hermes Agent**, **OpenAI Codex**, **Windsurf**, and **Context7** so AI agents generate zero-error Wolin integrations on the first pass.

---

## Repository Contents

| Path | Description |
| :--- | :--- |
| [`SKILL.md`](./SKILL.md) | Canonical agent skill workflow, live endpoints, local `--in-memory` & `$0` Free Tier rules, pitfalls, and verification gates. |
| [`references/ns-schema.md`](./references/ns-schema.md) | Official `https://wolin.id/ns#` RDF vocabulary (`avatarGlbUrl`, `avatarVrmUrl`, `passportTheme`, `worldLinks`, `wearableItem`), Turtle & JSON-LD examples, 60s HMAC-SHA256 `302` R2 asset offload, and ~180 reserved handle + Tombstone policies. |
| [`references/lego-blocks.md`](./references/lego-blocks.md) | Architecture & specification for the 4 Wolin Metaverse Lego Blocks (`<WolinAuthAndAvatar />`, `<MediaStageScreen />`, `<WolinAdSlot />`, `public/client-id.jsonld`). |

---

## Live Production Endpoints & Zero-Cost Sandbox

- **Solid-OIDC Provider & Passport (`https://auth.wolin.id`)**: Cloudflare Workers + D1, AWS SES v2 Magic Link, GitHub OAuth live (+ Google, Kakao, Apple, Naver, Discord adapters ready for credentials), 6-language i18n (`en/zh/es/pt/ja/ko`), Rybbit analytics + zero-PII D1 error/feedback loop, and `/assets/:pod_id/*` signed R2 streaming.
- **Sovereign Pod Data Plane (`https://pod.wolin.id` & `https://{handle}.wolin.id/`)**: `<30MB` Rust engine (`wolin-server`), Per-Pod Oxigraph RocksDB + bounded LRU (`256`) + `1 GiB` `$0` Free Tier quota, ~180 reserved handles + permanent Tombstone (`410 Gone`), short-lived `HMAC-SHA256` `302` R2 asset offload, and `JwksSignatureVerifier` + DPoP `at+jwt` verification.
- **Development Sandbox (No Public Devnet)**: Run `wolin-server --in-memory` locally (`http://127.0.0.1:4000`, ~6 ms startup) for offline/CI testing, or use the **`$0` Free Tier (`1 GiB`)** directly on `https://auth.wolin.id` and `https://pod.wolin.id`.

---

## Official SDKs & Starter Ecosystem

| Platform | Package | Repository |
| :--- | :--- | :--- |
| **TypeScript / React** | `@wolin-id/client`, `@wolin-id/react` | [`wolin-id/sdk-js`](https://github.com/wolin-id/sdk-js) |
| **Python 3.11+** | `wolin-id` | [`wolin-id/sdk-python`](https://github.com/wolin-id/sdk-python) |
| **Unity C#** | `id.wolin.unity` | [`wolin-id/sdk-unity`](https://github.com/wolin-id/sdk-unity) |
| **Swift 6 (iOS/macOS/visionOS)** | `WolinKit` | [`wolin-id/sdk-swift`](https://github.com/wolin-id/sdk-swift) |
| **Kotlin / Android / JVM** | `id.wolin:sdk-kotlin` | [`wolin-id/sdk-kotlin`](https://github.com/wolin-id/sdk-kotlin) |
| **3D Metaverse Starter** | `@wolin-id/metaverse-starter` | [`wolin-id/metaverse-starter`](https://github.com/wolin-id/metaverse-starter) |

---

## Installation

### Claude Code / Vercel Skills CLI / Cursor

```bash
npx skills add wolin-id/skills
```

### Hermes Agent

Clone or copy this directory under `~/.hermes/skills/wolin/` and load with `skill_view(name="wolin")`.

---

## Core Protocol Guardrails Encoded in This Skill

1. **Origin-Bound Solid-OIDC `client-id.jsonld`**: `new URL(client_id).origin === new URL(redirect_uri).origin` — zero API keys or app review required.
2. **RFC 7636 PKCE (`S256`) + RFC 9449 ES256 `DPoP` + `at+jwt`**: `htu` stripped of `?query` and `#fragment`, mandatory `ath` SHA-256 Base64URL hash, `typ: "at+jwt"` access tokens (blocks `id_token` confusion), and automatic `DPoP-Nonce` challenge retry.
3. **LDP Container Trailing Slash (`/`)**: Container IRIs always end with `/`.
4. **Easy UX**: End-user screens never display raw protocol jargon (`DPoP`, `JWK`, `ETag`, `RDF`, `Turtle`).
5. **MRC 3D Spatial Ad Viewability**: Frustum + front-facing + `>= 5%` projected screen area + unobstructed `Raycaster` line-of-sight + `1.5s` continuous active gaze, Zero-PII payload.

---

## License

Dual-licensed under either of:

- **MIT License** ([`LICENSE-MIT`](./LICENSE-MIT))
- **Apache License, Version 2.0** ([`LICENSE-APACHE`](./LICENSE-APACHE))

at your option. Copyright (c) 2026 Entropy Paradox, Inc. and Wolin Contributors.
