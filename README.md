# Wolin Agent Skills (`wolin-id/skills`)

> Official AI Coding Agent Skills & Protocol References for **Wolin (`https://wolin.id`)** — Sovereign Solid Pods, Passwordless Solid-OIDC + DPoP Identity, `https://wolin.id/ns#` 3D Avatar Interop, and Zero-PII Spatial AdSense.

Designed for **Claude Code**, **Cursor**, **Hermes Agent**, **OpenAI Codex**, **Windsurf**, and **Context7** so AI agents generate zero-error Wolin integrations on the first pass.

---

## Repository Contents

| Path | Description |
| :--- | :--- |
| [`SKILL.md`](./SKILL.md) | Canonical agent skill workflow, invariants, pitfalls, and verification gates. |
| [`references/ns-schema.md`](./references/ns-schema.md) | Official `https://wolin.id/ns#` RDF vocabulary (`avatarGlbUrl`, `avatarVrmUrl`, `passportTheme`, `worldLinks`, `wearableItem`), Turtle & JSON-LD examples, R2 private asset rules, and handle policies. |
| [`references/lego-blocks.md`](./references/lego-blocks.md) | Architecture & specification for the 4 Wolin Metaverse Lego Blocks (`<WolinAuthAndAvatar />`, `<MediaStageScreen />`, `<WolinAdSlot />`, `public/client-id.jsonld`). |

---

## Installation

### Claude Code / Vercel Skills CLI / Cursor

```bash
npx skills add wolin-id/skills
```

### Hermes Agent

Clone or copy this directory under `~/.hermes/skills/wolin/` and load with `skill_view(name="wolin")`.

### Manual Context Attachment

Reference `SKILL.md`, `references/ns-schema.md`, and `references/lego-blocks.md` directly in your IDE or agent workspace when building on:
- [`wolin-id/metaverse-starter`](https://github.com/wolin-id/metaverse-starter) (React 19 + Vite 6 + Three.js / R3F)
- [`wolin-id/sdk-js`](https://github.com/wolin-id/sdk-js) (`@wolin-id/client`, `@wolin-id/react`)
- `wolin-id/sdk-unity`, `wolin-id/sdk-swift`, `wolin-id/sdk-kotlin`, `wolin-id/sdk-python`

---

## Core Protocol Guardrails Encoded in This Skill

1. **Origin-Bound Solid-OIDC `client-id.jsonld`**: `new URL(client_id).origin === new URL(redirect_uri).origin` — zero API keys or app review required.
2. **RFC 7636 PKCE (`S256`) + RFC 9449 ES256 `DPoP`**: `htu` stripped of `?query` and `#fragment`, with automatic `DPoP-Nonce` challenge retry.
3. **LDP Container Trailing Slash (`/`)**: Container IRIs always end with `/`.
4. **Easy UX**: End-user screens never display raw protocol jargon (`DPoP`, `JWK`, `ETag`, `RDF`, `Turtle`).
5. **MRC 3D Spatial Ad Viewability**: Frustum + front-facing + `>= 5%` projected screen area + unobstructed `Raycaster` line-of-sight + `1.5s` continuous active gaze, Zero-PII payload.

---

## License

Dual-licensed under either of:

- **MIT License** ([`LICENSE-MIT`](./LICENSE-MIT))
- **Apache License, Version 2.0** ([`LICENSE-APACHE`](./LICENSE-APACHE))

at your option. Copyright (c) 2026 Entropy Paradox, Inc. and Wolin Contributors.
