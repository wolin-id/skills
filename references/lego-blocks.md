# Wolin (月印千江) Metaverse 4 Lego Blocks Specification

Reference architecture and implementation contract for the 4 modular building blocks in [`wolin-id/metaverse-starter`](https://github.com/wolin-id/metaverse-starter).

---

## Block 1: `<WolinAuthAndAvatar />` (`src/blocks/WolinAuthAndAvatar.tsx`)

### Purpose
One-click passwordless Wolin Passport sign-in paired with automatic cross-world 3D avatar loading from `https://wolin.id/ns#avatarGlbUrl`.

### Contract
1. **Passwordless Solid-OIDC + DPoP (`https://auth.wolin.id`)**:
   - Initiates PKCE `S256` authorization against `https://auth.wolin.id` (6-language i18n `en/zh/es/pt/ja/ko`, AWS SES Magic Link + GitHub OAuth live + 5 social providers ready) using `<origin>/client-id.jsonld`.
   - Exchanges `?code=...&state=...` with an ES256 `dpop+jwt` proof for a DPoP-bound `at+jwt` access token and verifies `<webId> solid:oidcIssuer <https://auth.wolin.id>`.
   - Supports zero-cost testing via local `wolin-server --in-memory` or the `$0` Free Tier (`1 GiB`) on `https://auth.wolin.id` / `https://pod.wolin.id`.
2. **3D Avatar Resolution Order**:
   - If `wolin:avatarGlbUrl` is present on the WebID profile and reachable over HTTPS, load via Three.js `GLTFLoader`, normalize bounding-box height to `~1.85m`, and align feet to `y = 0`.
   - If `wolin:avatarGlbUrl` is absent, offline, or fails CORS/network fetch, render a **procedural low-poly stylized 3D Explorer avatar** tinted by `wolin:passportTheme` so the scene never crashes or renders an empty origin.
3. **Easy UX Rule**:
   - End-user UI displays human concepts (*"Continue with Wolin"*, *"Passport"*, `luna.wolin.id`), never `DPoP`, `JWK`, `RDF`, `ETag`, or `OIDC`.

---

## Block 2: `<MediaStageScreen />` (`src/blocks/MediaStageScreen.tsx`)

### Purpose
3D amphitheater screen and spatial audio block designed around Wolin's **3-Tier Zero-Deficit Media Architecture**:

| Tier | Use Case | Transport | Server Bandwidth Cost |
| :--- | :--- | :--- | :--- |
| **Tier 1** | One-way Concerts, Keynotes, Lectures, Streams | YouTube Live (`youtube-nocookie.com/embed/...`) or CHZZK Live (`chzzk.naver.com/live/...`) inside 3D `<Html transform>` IFrame / native WebView | **$0** (carried by YouTube / Naver CDN) |
| **Tier 2** | 1:1 – 4-Person Proximity Voice / Small Rooms | WebRTC P2P (`RTCPeerConnection` + public STUN `stun:stun.l.google.com:19302`) | **$0** (peer-to-peer mesh) |
| **Tier 3** | 5+ Person Low-Latency Interactive Rooms | Cloudflare Realtime SFU or LiveKit (`Wolin` acts strictly as WebID permission token bridge; funded via Creator Stream Boost or World BYOK) | Metered to room sponsor / BYOK |

### URL Sanitization Rules
- Accept only `https:` or `http:` URLs; reject `javascript:`, `data:`, and embedded credentials.
- Validate YouTube video IDs (`^[a-zA-Z0-9_-]{6,20}$`) and CHZZK channel IDs (`^[a-zA-Z0-9_-]{6,64}$`) before constructing `<iframe src="...">`.

---

## Block 3: `<WolinAdSlot />` (`src/blocks/WolinAdSlot.tsx`)

### Purpose
Zero-PII sovereign 3D Spatial AdSense billboard extending 2D MRC Viewability standards into 3D WebGL worlds.

### 3D Viewability Verification Gate (`useSpatialAdViewability`)
A frame counts as `isViewable = true` **only** when all five checks pass simultaneously:

1. **Camera Frustum Intersection**: Billboard bounding sphere/quad intersects `THREE.Frustum` derived from `camera.projectionMatrix * camera.matrixWorldInverse`.
2. **Front-Facing Orientation**: `slotWorldNormal.dot(directionToCamera) >= 0.15` (back-of-billboard and steep grazing angles are rejected).
3. **Readable Distance**: `0.25m < distance <= maxDistance` (default `28m`).
4. **Screen-Area Share `>= 5%` (`0.05`)**: Project the 4 world-space quad corners into NDC (`[-1, 1]`), clamp to viewport, and compute projected screen-space area fraction (`>= 0.05` of total viewport area).
5. **Raycast Line-of-Sight (Zero Occlusion)**: Cast `THREE.Raycaster` from `camera.position` toward the billboard center (`far = distance - 0.12`) against visible scene meshes (excluding the billboard's own children and floor/grid helpers marked `userData.ignoreAdOcclusion = true`). Any blocking wall/prop sets `occluded = true`.

### Continuous Dwell & Anti-Fraud Rules
- **1.5s Continuous Gaze**: Accumulate `continuousDwellMs` across frames; reset to `0` immediately on any single occluded or `< 5%` area frame.
- **Active Input Gate**: Pause/reset accumulation if no pointer, wheel, or keyboard activity occurred within the last `30s` (prevents AFK farming).
- **Zero-PII Payload**: Gaze tracking runs strictly in local volatile memory. When `continuousDwellMs >= 1500`, emit once per creative cycle:
  ```ts
  {
    slotId: 'plaza-east-billboard-01',
    adId: 'house-wolin-passport-v1',
    worldContext: 'wolin-lunar-plaza',
    dwellMs: 1512,
    screenAreaRatio: 0.0842,
    distance: 6.14,
    impressionToken: 'imp_<one-time-uuid>',
    timestamp: 1763168400000
  }
  ```
  Never include `webId`, IP hashes, cookies, or cross-world user identifiers.
- **0% Empty-Slot Backfill**: Default to the built-in Wolin Passport procedural `CanvasTexture` House Ad whenever no external campaign creative is bound.

---

## Block 4: `public/client-id.jsonld` (Solid-OIDC Client ID Document)

### Purpose
Eliminates OAuth developer portal registration and static API keys. Hosting this static JSON-LD file on your world's HTTPS origin makes your domain an instant Wolin-compatible Relying Party.

### Template (`public/client-id.jsonld`)

```json
{
  "@context": [
    "https://www.w3.org/ns/solid/oidc-context.jsonld"
  ],
  "client_id": "https://world.example.com/client-id.jsonld",
  "client_name": "Your 3D Metaverse World",
  "client_uri": "https://world.example.com/",
  "redirect_uris": [
    "https://world.example.com/",
    "https://world.example.com/auth/callback"
  ],
  "post_logout_redirect_uris": [
    "https://world.example.com/"
  ],
  "scope": "openid webid offline_access",
  "grant_types": [
    "authorization_code",
    "refresh_token"
  ],
  "response_types": [
    "code"
  ],
  "token_endpoint_auth_method": "none",
  "application_type": "web",
  "dpop_bound_access_tokens": true
}
```

### Mandatory Security Invariant
`new URL(client_id).origin` **must** equal `new URL(redirect_uri).origin`. Wolin Cloud enforces origin binding to prevent arbitrary `client_id` spoofing.
