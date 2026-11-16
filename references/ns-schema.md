# Wolin (月印千江) Official RDF Vocabulary (`https://wolin.id/ns#`)

Authoritative schema reference for interoperable Wolin Passports, Sovereign Solid Pods, and cross-world 3D Metaverse applications.

- **Namespace IRI**: `https://wolin.id/ns#`
- **Recommended Turtle Prefix**: `wolin:`
- **Primary Profile Subject**: `https://{pod_id}.wolin.id/profile/card#me`

---

## 1. Prefix Declarations

```turtle
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
@prefix vcard: <http://www.w3.org/2006/vcard/ns#> .
@prefix solid: <http://www.w3.org/ns/solid/terms#> .
@prefix pim:   <http://www.w3.org/ns/pim/space#> .
@prefix ldp:   <http://www.w3.org/ns/ldp#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix wolin: <https://wolin.id/ns#> .
```

---

## 2. Core `https://wolin.id/ns#` Predicates

| Term | Full IRI | Object Kind | Cardinality | Description |
| :--- | :--- | :--- | :--- | :---
| `wolin:avatarGlbUrl` | `https://wolin.id/ns#avatarGlbUrl` | IRI (`https://...`) | `0..1` | Canonical glTF 2.0 binary (`.glb`) 3D avatar URL used by WebGL/Three.js, Unity, and Unreal worlds. |
| `wolin:avatarVrmUrl` | `https://wolin.id/ns#avatarVrmUrl` | IRI (`https://...`) | `0..1` | Optional VRM 0.x / 1.0 humanoid avatar URL with blendshapes and spring bones for VTuber/social worlds. |
| `wolin:passportTheme` | `https://wolin.id/ns#passportTheme` | `xsd:string` literal | `0..1` | Visual theme token for Passport cards & procedural fallback avatars (e.g. `"lunar-mercury"`, `"obsidian-cyan"`, `"aurora-violet"`). |
| `wolin:worldLinks` | `https://wolin.id/ns#worldLinks` | IRI (`https://...`) | `0..*` | Deep-link URLs to metaverse spaces or rooms where the user is active or hosts events (Organic Discovery). |
| `wolin:wearableItem` | `https://wolin.id/ns#wearableItem` | IRI or `xsd:string` | `0..*` | Portable cross-world wearable item identifier or signed Pod resource IRI. |

### Companion W3C Solid Predicates (Required for Interop)

| Term | Full IRI | Role |
| :--- | :--- | :--- |
| `solid:oidcIssuer` | `http://www.w3.org/ns/solid/terms#oidcIssuer` | Authoritative IdP trust anchor (canonical `<https://auth.wolin.id>`). Must match token `iss`. |
| `pim:storage` | `http://www.w3.org/ns/pim/space#storage` | Root LDP Storage Container (`<https://{pod_id}.wolin.id/>`, always trailing `/`). |
| `ldp:inbox` | `http://www.w3.org/ns/ldp#inbox` | Linked Data Notifications inbox container. |
| `foaf:name` / `vcard:fn` | `http://xmlns.com/foaf/0.1/name` | Human-readable display name shown on Passport & 3D overhead nameplate. |

---

## 3. Canonical WebID Profile Example (`text/turtle`)

Served at `GET https://luna.wolin.id/profile/card` with `Accept: text/turtle`:

```turtle
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
@prefix vcard: <http://www.w3.org/2006/vcard/ns#> .
@prefix solid: <http://www.w3.org/ns/solid/terms#> .
@prefix pim:   <http://www.w3.org/ns/pim/space#> .
@prefix ldp:   <http://www.w3.org/ns/ldp#> .
@prefix wolin: <https://wolin.id/ns#> .

<#me>
  a foaf:Person ;
  foaf:name "Luna Choi" ;
  vcard:fn "Luna Choi" ;
  solid:oidcIssuer <https://auth.wolin.id> ;
  pim:storage <https://luna.wolin.id/> ;
  ldp:inbox <https://luna.wolin.id/inbox/> ;
  wolin:avatarGlbUrl <https://cdn.wolin.id/pods/luna/avatar.glb> ;
  wolin:avatarVrmUrl <https://cdn.wolin.id/pods/luna/avatar.vrm> ;
  wolin:passportTheme "lunar-mercury" ;
  wolin:worldLinks <https://plaza.wolin.id/>, <https://concert.example.com/room/lunar> ;
  wolin:wearableItem "crescent-halo-mk1", <https://luna.wolin.id/inventory/jacket-01.ttl#item> .
```

---

## 4. JSON-LD Context Mapping

When consuming or emitting JSON-LD documents:

```json
{
  "@context": {
    "foaf": "http://xmlns.com/foaf/0.1/",
    "solid": "http://www.w3.org/ns/solid/terms#",
    "pim": "http://www.w3.org/ns/pim/space#",
    "wolin": "https://wolin.id/ns#",
    "avatarGlbUrl": { "@id": "wolin:avatarGlbUrl", "@type": "@id" },
    "avatarVrmUrl": { "@id": "wolin:avatarVrmUrl", "@type": "@id" },
    "passportTheme": "wolin:passportTheme",
    "worldLinks": { "@id": "wolin:worldLinks", "@type": "@id" },
    "wearableItem": "wolin:wearableItem"
  }
}
```

---

## 5. Binary Asset Delivery & Access Control Boundary

1. **Public Avatar & World Assets (`$0` Egress)**:
   - Public `.glb` / `.vrm` avatars and textures are served directly from Cloudflare R2 + Edge CDN (`https://cdn.wolin.id/...`) with `Access-Control-Allow-Origin: *` for WebGL `GLTFLoader`.
2. **Private / Authorized Binary Assets (`302` HMAC-SHA256 Offload)**:
   - Never expose raw R2 buckets for private inventory items.
   - `wolin-server` validates the caller's `DPoP` proof + JWKS `at+jwt` access token + `.acl` / ACP policy first, then issues a **short-lived (`60`-second) `HMAC-SHA256` (`wolin-asset-v1`) `302 Found` redirect** (`>= 256 KiB` `NonRDFSource`) to `https://auth.wolin.id/assets/{pod_id}/{path}?exp=...&sig=...` so authorization remains fail-closed while binary bandwidth streams directly from Cloudflare R2.
3. **Subdomain Handle (`{pod_id}.wolin.id`) Constraints**:
   - `$0` Free Tier (`1 GiB` quota): `4–32` chars, lowercase ASCII `a-z`, `0-9`, `-` (no leading/trailing `-`, no `--`, no `xn--` punycode).
   - `3`-char rare handles: Pro/Creator tier only (engine syntax allows `3–32` chars).
   - `1–2`-char handles & ~180 built-in RFC 2142 / DNS / OAuth / infrastructure / brand handles: blocked in both `wolin-server` and `wolin-cloud`. Deleted `pod_id` handles enter permanent `tombstone` freeze (`410 Gone`) to prevent identity hijacking.
