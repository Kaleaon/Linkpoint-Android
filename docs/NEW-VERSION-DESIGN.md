# Linkpoint — New Version Design

**Status:** Design study, 2026-11-25. Written at Joe's order ("start understanding
linkpoint-design and react-linkpoint and linkpoint to fully design the new version").
No code was changed by this study. Every decision below is marked **DECIDED**
(with its reason) or **OPEN** (with what would decide it).

**Sources studied (all read 2026-11-25):** the three local checkouts and their
GitHub state — `Kaleaon/linkpoint-design` (local: `~/workspace/linkpoint-design-work/repo`),
`Kaleaon/React-Linkpoint` (local: `~/workspace/react-linkpoint`, checkout of
2026-10-04, behind GitHub main), `Kaleaon/Linkpoint` (local: `~/workspace/linkpoint-agy`,
checkout of 2026-10-09, behind GitHub main) — plus `Kaleaon/Linkpoint-stitch`
(GitHub only), the `secondlife_linkpoint_design` skill, and the Lumiya
modernization findings in `~/workspace/lumiya-repair/` (`FILAMENT-SETUP.md`,
`P2-AUDIT.md`, `AIS3-DESIGN.md`, `P3-NOTES.md`). The Lumiya renderer-decision
and XR notes were still being written by parallel sessions and are not cited.

---

## 1. Design principles

1. **Features come from the current viewers, as specs — never as C++ ports.**
   (Joe's directive, 2026-11-25.) The feature set and behaviour of the new
   version are specified from the **official Second Life viewer** and
   **Firestorm** first — Firestorm carries features the official viewer lacks
   (RLVa is the standing example) — with **Alchemy** and **Kokua** consulted
   where they lead (OpenSim behaviour, lean performance). Each feature is
   captured as a spec (wire behaviour, UX behaviour, edge cases) from the
   named viewer's current source/docs, then implemented fresh in this
   project's modern stack. Anything that exists today only as C++ (in the
   viewers, or in any legacy tree) is **re-implemented in Kotlin/TypeScript**;
   no C++ is ported and no new C++ is written unless a named native-only API
   forces a thin layer (the Filament and WebRTC native libraries are consumed
   as vendor binaries, which is not project C++). Per-feature source citations
   are in §8; the spec-capture step re-verifies each citation against the
   named viewer's then-current source before implementation starts.
2. **One design truth.** `linkpoint-design` is the single source for layout
   geometry and Ktheme colour. No screen hardcodes what a token owns (§7).
3. **Protocol truth comes from the wire, not from memory.** Message numbers
   and layouts are taken from the official viewer's current
   `message_template.msg` and capability behaviour (the Lumiya P2 audit found
   the master template can be stale against the viewer copy — the viewer copy
   wins). OpenSimulator source is the reference for OpenSim divergences.
4. **Verification is a feature.** Nothing is claimed working without the level
   of proof stated next to it (unit / mock grid / real OpenSim / live SL /
   on-device). Both predecessor codebases carry "never run against a live
   grid / never run on a device" debt; the new version retires that debt
   first, not last (§10, Phase 0).
5. **Lumiya is the behavioural oracle, not a code source.** Lumiya/Lumiya-Redux
   behaviour (and the repaired Lumiya build's audit trail) specifies what a
   mobile SL client must survive; code is written fresh, honouring the
   provenance rules already used by both Linkpoint projects.
6. **TPV compliance is load-bearing.** Linden Lab Third-Party Viewer policy:
   honest viewer identity/channel strings, no credential handling through
   third-party proxies, encrypted credential storage only.

## 2. Product definition

**What it is:** Linkpoint, new version — a mobile-first, fully rendering
Second Life and OpenSimulator viewer for Android, with a web companion. It
logs in, renders the world (terrain, prims, mesh, sculpts, avatars with Bakes
on Mesh, EEP sky/water, PBR materials), and covers the social core (chat, IM,
groups, friends, inventory, teleports) to current-viewer parity.

**Who it is for:** Joe first — a working modern SL client on Android that is
his, open, and not abandoned (the Lumiya situation) — and, by construction,
any resident who wants a maintained open mobile viewer.

**How it differs from Lumiya:** Lumiya is a repaired 2019 closed-era binary
being modernized in place (decompiled smali, 32-bit legacy, Vivox). The new
Linkpoint is clean-room modern code (Kotlin/Compose/Filament, 64-bit from
birth) that takes Lumiya only as its behavioural reference — the same
relationship React-Linkpoint already documents. Lumiya's modernization
continues in parallel as the oracle and fallback client; it is not the new
version's codebase.

**How it differs from the current Linkpoint(s):** there are effectively three
Linkpoints today (§3). The new version ends the split: one native app, one
design system, one protocol core — instead of a React web app, a Rust/Tauri
overhaul, and a native Kotlin app evolving separately inside two repos.

## 3. What exists today (evidence summary)

| Project | What it actually is | State (2026-11-25) |
| --- | --- | --- |
| **linkpoint-design** (`Kaleaon/linkpoint-design`) | The design source of truth: interactive mockup of 20 screens × 6 layout packs × 24+ Ktheme colour packs × device sizes, Ktheme JSON schemas, a React port of the mockup, a Ktheme component/token project (`~/workspace/linkpoint-design-work/Ktheme`), and CI that turns React-Linkpoint changes into a design todo checklist daily. | GitHub main pushed **today**; **0 open PRs** (the Nov 23 PR sweep goal is complete on GitHub). Local checkout clean, on the merged fix branch. |
| **React-Linkpoint** (`Kaleaon/React-Linkpoint`) | Two apps in one repo: (a) a TypeScript/React PWA + Electron desktop viewer over `@caspertech/node-metaverse` with a hand-written WebGL2 renderer (`src/linkpoint/`, `core/*.cjs`); (b) **`android-kotlin/`** — a standalone native Kotlin + Jetpack Compose + **Filament** viewer with its own from-scratch protocol core, a runnable mock grid, and an automated real-OpenSim test rig. | GitHub main active (PR **#420** merged 2026-11-24; open: #425, #401, #399). Local checkout is 2026-10-04 — behind GitHub. Web app: login/MFA/chat/dialogs/teleport done against stubs; **never run against a live grid**; renderer works in headless Chromium only. `android-kotlin`: 210 core tests, verified end-to-end against **real OpenSim 0.9.3** (login, teleports, region crossing, chat/IM, inventory, sounds, 15k-object OAR stress); **never run against Second Life**; Filament view compiles but **has never run on a physical device**. |
| **Linkpoint** (`Kaleaon/Linkpoint`) | The umbrella repo. Active direction per its README/OVERHAUL docs: "Linkpoint Next" — React UI (`apps/linkpoint`) over a **Rust core** (`crates/linkpoint-{core,protocol,assets,scene,cache,tauri}`), Tauri shell, `packages/design-system` synced from linkpoint-design; the original Kotlin/Android app survives under `legacy/Linkpoint` as reference. 86 Rust files; foundational tests; Tauri shell **fails closed** (no production transport yet). | GitHub main last merged 2026-11-18 (#924); **37 open PRs**, mostly stale Stitch rebases. Local clone is a detached, dirty worktree mid-rebase (one unmerged conflict) from the October agy sessions. |
| **Linkpoint-stitch** (`Kaleaon/Linkpoint-stitch`) | AI-Studio-born Kotlin/Compose implementation of the design system (`LinkpointTheme.kt`, cockpit screen, Pass-2 HUD composition per the design skill). | GitHub only (pushed 2026-11-24); no local checkout. Treated as a design-implementation reference, not a codebase candidate. |

**How the three relate (the one-paragraph truth):** linkpoint-design is the
upstream design authority for both apps (React-Linkpoint syncs it via
`npm run sync-design`; Linkpoint syncs 24 themes into `packages/design-system`).
React-Linkpoint is where working viewer behaviour has actually been proven —
in TypeScript against stubs, and, inside the same repo, in native Kotlin
against a real OpenSim. Linkpoint (the repo) is an in-flight architectural
overhaul that imports React-Linkpoint's screens onto a Rust core that has not
yet carried a production session. The three are one product at three levels of
maturity — design, proven behaviour, and an unfinished re-platforming — and
the new version consolidates them rather than adding a fourth.

## 4. Platform and stack decision

**DECIDED: native Android first — Kotlin + Jetpack Compose + Filament —
grown from React-Linkpoint's `android-kotlin/` lineage.**
Reasons, from the evidence:

- It is the only candidate that has ever completed a real session against a
  real simulator (OpenSim 0.9.3, automated, repeatable), including the hard
  parts: circuit reliability, region crossing, teleport races, asset decode
  at 15k-object scale, and a JPEG2000 decoder verified bit-exact against
  OpenJPEG. The Rust core has never carried a production transport (its own
  progress doc says the shell fails closed); the TypeScript app has never
  touched a live grid.
- It already embodies today's renderer decision: Filament (matching the
  Lumiya programme's Filament findings — SL PBR *is* glTF metallic-roughness,
  Filament's native model; AARs ship arm64 + 16 KB-aligned libs), and it
  already shares `.filamat` materials with the web renderer via one pinned
  matc version.
- Kotlin is Joe's platform language and the stack the no-C++ directive points
  at ("Kotlin et al."). Compose consumes the Ktheme design system directly
  (the Linkpoint-stitch Compose work proves the mapping).
- Its gaps are enumerable and honest (its README lists them); the other
  candidates' gaps include their foundations.

**DECIDED: the React/TypeScript lineage continues as the web companion, not
the app.** The PWA/Electron build remains the browser-reachable client
(login/caps/metadata today, desktop UDP via Electron), keeps consuming the
same design system and the same shared Filament materials, and donates its
proven TypeScript behaviour specs (EEP/atmosphere maths — already used as
the numeric reference for the Kotlin env code — interaction/dialog handling,
contacts/calendar) to the native app as specs and test vectors.

**DECIDED: the Rust/Tauri overhaul in `Kaleaon/Linkpoint` is not the new
version's base.** Its contracts are worth mining (the `ViewerClient`
command/event boundary, Rust-generated TypeScript contract, renderer-neutral
scene model are good ideas and are adopted as *design patterns* in §5), but
adopting the Rust core would restart protocol work that the Kotlin core has
already finished and verified, and would split Joe's maintenance across two
native stacks. The legacy Kotlin app under `legacy/Linkpoint` stays what it
already is: reference.

**DECIDED: the new version's repo home is a new, purpose-built repo —
`Kaleaon/Linkpoint-Android`** (Joe's call, 2026-11-25, choosing option (c)
of the original open item: (a) promote `React-Linkpoint/android-kotlin` in
place, (b) a `Kaleaon/Linkpoint` adoption branch, (c) a new repo). The
`android-kotlin/` tree from React-Linkpoint GitHub main `67ea26df` was
imported as the new repo's trunk on 2026-11-25 — structure preserved at the
repo root, every blob verified against the source tree, provenance recorded
in the repo's `docs/PROVENANCE.md`. The React web companion stays in
React-Linkpoint; the design authority stays `Kaleaon/linkpoint-design`.
Pushes to `Kaleaon/Linkpoint-Android` go through the PAT + Git Database API
route, like every GitHub write from this workspace.

**Apple platforms: a sibling founding repo, `Kaleaon/Linkpoint-Apple`,**
was created the same day (Joe, 2026-11-25) as the home for Linkpoint on
Apple platforms — iOS + macOS first, visionOS in view given the project's
OpenXR/XR direction. It holds no app code yet by design: its first design
task is the platform stack decision, which remains **OPEN** — native
Swift/SwiftUI (+ Metal or Filament's iOS builds) versus a Kotlin
Multiplatform core shared with the Android app (whose `core/` is pure JVM
and Android-free, so sharing is a real candidate). iOS remains out of the
Android v1 scope; it now has a home and a named first task instead of
waiting on the Android live-grid gate.

## 5. Architecture

Layered, with dependencies pointing inward (adopting the Linkpoint overhaul's
one good rule): UI never touches packets; the session core never touches
pixels.

```
┌─────────────────────────────────────────────────────────────┐
│ Compose UI (design-system tokens only; 6 layout packs)      │
├─────────────────────────────────────────────────────────────┤
│ Viewer state projection (StateFlow stores: chat, friends,   │
│ inventory, groups, region, scene, environment, offers)      │
├─────────────────────────────────────────────────────────────┤
│ Session core (Kotlin, pure JVM): login/MFA, circuit, caps,  │
│ event queue, AIS3 inventory, chat/IM, teleports, RLVa       │
├─────────────────────────────────────────────────────────────┤
│ Scene model (renderer-neutral: objects, linksets, avatars,  │
│ terrain, particles, environment frames)                     │
├─────────────────────────────────────────────────────────────┤
│ Asset pipeline: caps fetch → cache → decoders (LLMesh,      │
│ sculpt, JPEG2000, .anim, baked/PBR material assets)         │
├─────────────────────────────────────────────────────────────┤
│ Renderer (Filament): scene sync, materials (.filamat),      │
│ Pass-1 world + Pass-2 HUD overlay, XR-ready view management │
└─────────────────────────────────────────────────────────────┘
```

- **Core (`core/` lineage from android-kotlin):** packet codec and message
  definitions generated/checked against the official `message_template.msg`;
  `Circuit` (UDP reliability, zero-coding, ACK batching); `Caps` +
  EventQueue; `ViewerSession` exposing StateFlows. Pure JVM, unit- and
  mock-grid-tested, no Android dependencies — this is what makes the core
  reusable by the web companion's tests and any future iOS effort.
- **Scene model:** normalized, renderer-neutral scene state (the overhaul's
  `linkpoint-scene` idea, implemented in Kotlin): object/linkset lifecycle,
  avatar state (skeleton, baked textures, animations), terrain composition,
  EEP environment frames sampled by time. The renderer consumes deltas; the
  protocol layer never builds geometry.
- **Renderer:** Filament, per §6. World rendering is Pass 1 (perspective);
  scripted HUD attachments (slots 31–38) render in Pass 2 (orthographic
  overlay in `HudSafeRect`, hit-tested before camera drag) — per the design
  system's cockpit rules (§7).
- **Shell:** Android app module — Compose UI, foreground service for the
  session, Keystore-backed secret storage (the android-kotlin `SecretStore`
  AES-GCM pattern, including its encrypted MFA-hash storage), cleartext-HTTP
  policy per grid (OpenSim needs it; SL grids are HTTPS).
- **Test rigs are part of the architecture,** inherited from android-kotlin:
  the runnable mock grid, the automated real-OpenSim harness
  (`tools/opensim/live.py` pattern), and a counts-only live-SL smoke test
  that never logs resident data.

## 6. Rendering

**DECIDED: Filament** (currently pinned at 1.53.4 in android-kotlin's
materials pipeline; the Lumiya programme has 1.81.0 installed and
smoke-tested — **OPEN:** the exact version to standardize on, decided by a
compatibility pass; the rule from android-kotlin stands: matc and runtime
versions must match across web and native, because they share materials).

- **Materials:** `.mat` sources compiled offline by matc to `.filamat`;
  shared between the web renderer and the native app. The prim/particle/flat/
  sky/water/terrain set exists; PBR (glTF metallic-roughness: base colour,
  metallic, roughness, normal, emissive, occlusion) is the v1.x addition,
  fed by SL's PBR material assets and the per-face material overrides that
  arrive via GenericStreamingMessage method 0x4175 (glTF override payloads —
  retained by the Lumiya protocol work and natively Filament-shaped).
- **Geometry:** decoded SL data (prim volume generator, LLMesh LODs, sculpt
  maps) feeds Filament VertexBuffer/IndexBuffer/RenderableManager directly —
  gltfio parses PBR *material assets*; it does not replace SL mesh decode
  (per FILAMENT-SETUP.md).
- **Avatars:** Bento skeleton (159 bones), GPU skinning, Bakes on Mesh
  compositing (the baker is compute work in the core, not the renderer),
  `.anim` playback with priority blending — specified from the official
  viewer's avatar system; the TypeScript avatar implementation in
  React-Linkpoint (skeleton/animation/skinning, verified in headless
  Chromium) is the working spec and test reference.
- **Environment:** EEP day-cycle sampling, sky scattering, water with
  Fresnel/reflection/glint — the TypeScript `eep.ts`/`atmosphere.ts` ports
  of the official viewer's maths are the numeric reference (already used to
  check the Kotlin env code number-for-number). Water/caustics advancement
  is a research track feeding this layer (a dedicated water-rendering skill
  is being authored in parallel for the Lumiya programme; its findings are
  an input, not a dependency).
- **Performance rules:** renderer object caps and draw-distance LOD are
  explicit settings (the android-kotlin stress run exposed unbounded GPU
  texture memory — eviction policy is a v1 requirement, not a later fix);
  frustum culling against per-mesh bounds; skinned objects excluded from
  culling until bounds are pose-aware.

## 7. UI and theming

Per linkpoint-design and the `secondlife_linkpoint_design` skill:

- **Geometry ⊥ colour.** Layout packs (structural geometry/navigation) and
  Ktheme palettes (colour mood) are orthogonal: any of the **6 layout packs**
  (Polished/Glass, Terminal, Sweep Console/LCARS, Metro, Rule & Rail, Press)
  pairs with any of the **24+ Ktheme palettes** (four families: Terminal &
  Neon, Console & Amber, Metal & Jewel, Daylight, plus community packs) at
  runtime, with no per-screen hex overrides.
- **Ktheme JSON is upstream.** Palettes conform to the app-neutral Ktheme
  schema (`metadata`, `darkMode`, `colorScheme`, optional `effects`,
  `adaptation`) and sync from linkpoint-design into the app via typed token
  adapters — the sync pipeline that already exists on both lineages
  (`sync-design` on React; `packages/design-system` on Linkpoint) is kept,
  re-pointed at the new app.
- **WCAG enforcement is automatic:** every semantic foreground token passes
  `ensureMinContrast` against both `bg` and `surf` (≥ 4.5:1 text, ≥ 3.0:1
  chrome), hue-preserving — on every palette, including the Daylight and
  deep-jewel packs where this has bitten before.
- **Cockpit ergonomics:** ≥ 85% of the mobile 3D viewport unobstructed;
  `WindowInsets.safeDrawing` honoured; 48dp touch targets; thumb-zone dock
  and thumbstick placement options; 1-tap Zen 3D mode. LCARS is built as
  real split-frame geometry on a pure-black void — never as recoloured
  cards.
- **Screens (20 in the design mockup):** login, chat, IM, groups, radar,
  map, inventory, profile, world/cockpit, design studio, system dialogs
  (permissions, teleport lures, pay, group notices) — the design dictionary's
  state contract (screen → session-state mapping) carries over unchanged.

## 8. Feature scope

Citations name the viewer each feature's spec comes from: **[Official]** =
official SL viewer, **[Firestorm]**, **[Alchemy]**, **[Kokua]**, **[Lumiya]**
= Lumiya/Lumiya-Redux behavioural oracle, **[LP]** = Linkpoint-original
(already built in a predecessor). Per Principle 1, each spec is re-verified
against the named source at implementation time.

### v1 (the consolidating release)

- Login incl. **MFA challenge + encrypted remembered hash**, grid list
  (Agni/Aditi/OpenSim grids), start locations, login redirects **[Official]**;
  android-kotlin already implements this shape (Keystore AES-GCM hash store).
- UDP circuit, caps, event queue; **AIS3 inventory mutations** (rename,
  delete, update, link, folder create) with the fallback policy proven in the
  Lumiya P2 work **[Official]**; inventory browse + orphan recovery
  **[Official]**.
- Local chat, IM, group chat, script/text-box dialogs, teleport lures and
  offers, inventory offers, group invites/notices, friend requests
  **[Official]** — the React lineage's interaction layer is the worked spec.
- Friends/presence, radar, world map + teleports (by name/home/lure, region
  crossing) **[Official]** — android-kotlin verified these against OpenSim.
- World rendering: terrain, prims (full profile/path), LLMesh, sculpts,
  JPEG2000, particles, EEP sky/water **[Official formats]**.
- Avatars: Bento skeleton, skinning, **Bakes on Mesh**, animation playback
  **[Official]**; shape sliders follow in v1.x (they are visual-param work,
  spec **[Official]**).
- HUD attachments: Pass-2 rendering + touch with face/UV hit results
  **[Official semantics; LP design]**.
- **RLVa**: parser + controller with enforcement at the point of action
  **[Firestorm]** (the official viewer ships no RLVa; Firestorm is the
  reference implementation, backed by the `secondlife_rlva` skill).
- Grid-synced mute list, people/group/place search, L$ balance +
  transactions + pay **[Official]**.
- Contacts + calendar export **[LP]** (already built twice; keep).
- Filament renderer with the shared material set; object caps/LOD settings.

### v1.x / later (explicitly not v1)

- **PBR materials** end-to-end (assets, overrides, probes) **[Official]**.
- **Voice over WebRTC** — signalling via `ProvisionVoiceAccountRequest` /
  `VoiceSignalingRequest` caps **[Official]**. (The React README's claim
  that SL voice "cannot be implemented by substituting ordinary WebRTC" is
  superseded: Linden Lab migrated SL voice to WebRTC, the android-kotlin
  README and the Lumiya P4 work both proceed on WebRTC, and the
  `secondlife_voice_webrtc` skill documents the signalling. Vivox is not
  carried into the new version in any form.)
- **XR/VR via OpenXR** — §9.
- Media on a Prim **[Official/Firestorm]**; texture upload + inventory
  editing/outfits; group chat polish; 360° snapshot **[Firestorm]**;
  area search and phototools **[Firestorm]**; avatar shape sliders;
  texture-animation/flexible prims; shadows/reflection probes tuning
  **[Official]**.
- iOS (OPEN, §4).

## 9. XR / VR

Joe's direction: "making sure VR works well is a plus."

**DECIDED: OpenXR is the only VR path.** No Cardboard/GVR anywhere in the new
version (the Lumiya programme is removing GVR from Lumiya in parallel and
replacing it with an OpenXR foundation; the new Linkpoint starts there).
Design consequences taken now, so VR is not bolted on later:

- The renderer's view management is XR-shaped from v1: per-eye views,
  a pose-driven camera rig separate from the orbit camera, and a scene
  model that does not assume a single 2D frame.
- Filament is the renderer on both paths (it is the Android XR ecosystem's
  reference PBR engine); stereo rendering is a renderer configuration, not
  a second renderer.
- The Pass-2 HUD/cockpit UI model maps to XR panels; the 2D cockpit ships
  first and its layout rules (unobstructed viewport, thumb zones) carry
  over as the comfort baseline.

**Phasing:** v1 ships flat (phone in hand) with the XR-ready architecture;
an OpenXR spike (session lifecycle, swapchain, per-eye present on a real
headset) is the first post-v1 milestone, gated on access to an XR device —
no VR behaviour is claimed before it runs on hardware (Principle 4).

## 10. Implementation roadmap

- **Phase 0 — Gates before features.** Live-SL smoke test (counts-only,
  Joe's test account) against the android-kotlin core; Filament renderer
  run on a physical Android device with frame-time numbers recorded;
  repo-home decision (§4) executed. *Exit: the two "never run" debts are
  retired or explicitly re-scoped by Joe.*
- **Phase 1 — Consolidation.** android-kotlin becomes the new version's
  trunk: design-system sync re-pointed from linkpoint-design; Compose
  screens rebuilt to the 20-screen design on Ktheme tokens (Linkpoint-stitch
  as the Compose reference); React web companion kept building against the
  same tokens/materials.
- **Phase 2 — Protocol parity.** AIS3 mutations + orphan recovery, remaining
  event wiring (offers/invites/notices), grid-synced mute, search,
  transactions/pay — each as a spec-capture from §8's cited viewer, with
  mock-grid + OpenSim tests before any live claim.
- **Phase 3 — People.** Avatars end-to-end: skinning on device, BoM baker,
  animation playback, attachments incl. rigged; HUD Pass-2 with face/UV
  touch. RLVa parser/controller with enforcement points.
- **Phase 4 — Materials & environment.** PBR end-to-end, EEP polish
  (real sun phase from `SimulatorViewerTime`, water normal maps, clouds),
  GPU memory eviction and LOD policy hardening from the stress rig.
- **Phase 5 — Voice & media.** WebRTC voice (caps signalling, native WebRTC
  via vendor binary), MoaP, streaming media.
- **Phase 6 — XR.** OpenXR spike → stereo world view → XR cockpit panels,
  on-device gated.
- **Throughout:** linkpoint-design remains upstream (design changes land
  there first and sync down); Lumiya's audits remain the mobile-behaviour
  oracle; every phase ends with the OpenSim harness green and a device
  record, not just CI green.

## 11. Carry-over and drop list

**From linkpoint-design:** everything — mockup, Ktheme schemas and palettes,
layout-pack specs, design dictionary contract, the sync tooling pattern.
It stays a separate upstream repo.

**From React-Linkpoint (web lineage):** the design-sync pipeline; EEP/
atmosphere/windlight maths and their test vectors; the interactions layer
(dialogs, lures) as specs; contacts/calendar; the mock-grid concept's
TypeScript twin; CI/release packaging know-how (incl. the Filament matc
caching). The hand-written WebGL renderer is **not** carried into the native
app (Filament replaces it); it remains the web build's renderer until/unless
the web companion adopts the shared-material Filament path via WebGPU/WebGL.

**From React-Linkpoint `android-kotlin/`:** the trunk — protocol core,
decoders (JPEG2000, LLMesh, sculpt, prim volume), session/state model,
mock grid + OpenSim harness, Compose app shell, Filament world renderer,
material sources, SecretStore. Its documented gaps become §8/§10 work items.

**From `Kaleaon/Linkpoint`:** the `ViewerClient` boundary pattern and
contract-generation idea (re-expressed in Kotlin/TS); the renderer-neutral
scene-model idea; threat-model and endpoint-policy documents. **Dropped:**
the Rust core and Tauri shell as the app base (kept in-repo as an archived
experiment); the Dart/C#/WASM package sprawl (no consumer ever materialized);
`legacy/Linkpoint` remains reference-only, as it already is.

**Dropped everywhere:** node-metaverse as a protocol dependency for the
native app (it remains only inside the legacy web build until that build's
own migration); the install-time `patch-metaverse.cjs` fragility with it;
Vivox; Cardboard/GVR; any C++ project code (Principle 1).

## 12. Open decisions (for Joe)

1. **Repo home** (§4) — **DECIDED 2026-11-25 (Joe): new purpose-built
   repos.** `Kaleaon/Linkpoint-Android` is the Android trunk (android-kotlin
   imported from React-Linkpoint @ `67ea26df`, provenance in its
   `docs/PROVENANCE.md`); `Kaleaon/Linkpoint-Apple` is the founding repo
   for Apple platforms, whose own first open item is its platform stack
   (native Swift vs Kotlin Multiplatform).
2. **Filament version standardization** — 1.53.4 (android-kotlin's pin) vs
   1.81.0 (installed for Lumiya). Decides: one compatibility pass over the
   shared materials.
3. **Avatar asset policy** — React-Linkpoint's self-imposed "no binary
   Lumiya assets" rule blocked its avatar work; android-kotlin instead
   extracts avatar geometry with scripts and bundles Lumiya's text
   animations. The new version needs one stated rule for avatar mesh/anim
   provenance (generate from visual-param definitions vs extract vs
   bundle-with-notice). Decides: Phase 3's starting point.
4. **Name/branding continuity** — "Linkpoint" plainly; version numbering
   starts where? (Cosmetic, but store listings and the login channel string
   — currently `Linkpoint Viewer` — depend on it.)
5. **Live-test account** — Phase 0's SL smoke test needs a designated test
   account (the harness exists and is counts-only; it has never been run).

## 13. Risks

- **Live-grid unknowns dominate.** Every predecessor defers them; SL may
  diverge from OpenSim in caps behaviour (AIS3 acceptance, MFA edge cases)
  in ways only Phase 0 can surface. Mitigation: Phase 0 is first, and the
  Lumiya P2 audit's unverifiable-without-login list is the checklist.
- **Filament on real hardware is unmeasured** in every Linkpoint lineage.
  Mitigation: Phase 0 device gate; object caps/LOD/eviction as v1 features.
- **Stale-branch gravity.** Linkpoint has 37 open PRs and React-Linkpoint a
  fast-moving Stitch pipeline; consolidation must freeze a baseline or it
  will chase both. Mitigation: Phase 1 starts from a named commit on each
  repo (React-Linkpoint GitHub main of 2026-11-24, #420 merge `67ea26df`,
  is the current candidate baseline for specs; android-kotlin's content
  must be re-diffed against GitHub main when Phase 1 starts, since the
  local checkout studied here is from 2026-10-04).
- **Design drift.** Two sync pipelines already exist; a third would fork
  the design truth. Mitigation: Principle 2 — one upstream, typed adapters
  only.
