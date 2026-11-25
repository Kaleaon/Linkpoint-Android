# Linkpoint Viewer for Android

The new version of Linkpoint — a native Android viewer for Second Life and
OpenSimulator. This repository is the new version's home (created
2026-11-25); the code trunk is the `android-kotlin/` project imported from
`Kaleaon/React-Linkpoint` (see `docs/PROVENANCE.md` for exactly what came
from where).

## Stack

- **Kotlin** — the whole app; the protocol core is pure JVM with no Android
  dependencies
- **Jetpack Compose** — UI, driven entirely by design-system tokens
- **Filament** — renderer (SL PBR is glTF metallic-roughness, Filament's
  native model); `.mat` sources in `materials/`, compiled `.filamat` in
  `app/src/main/assets`
- **OpenXR** — the only VR path (no Cardboard/GVR); XR-shaped view
  management from v1, the OpenXR spike is the first post-v1 milestone
- **WebRTC** — voice, via the `ProvisionVoiceAccountRequest` /
  `VoiceSignalingRequest` caps (no Vivox)
- **64-bit** throughout; no project C++ — modern viewer features are
  implemented fresh in Kotlin from the official viewer / Firestorm /
  Alchemy / Kokua as specs

Filament version note: the imported tree pins Filament 1.53.4 (its matc and
runtime must match, because materials are shared with the web renderer);
the Lumiya programme has 1.81.0 installed. Standardizing on one version is
an open item decided by a compatibility pass.

## Design authority

`Kaleaon/linkpoint-design` is the single source of truth for layout
geometry and Ktheme colour — no screen hardcodes what a token owns. The
new-version design study (2026-11-25) that this repo executes is copied at
`docs/NEW-VERSION-DESIGN.md`; its DECIDED list governs, its OPEN list is
Joe's calls.

## Status

Proven against a real simulator, not yet against Second Life or hardware:

- Verified end-to-end against **real OpenSim 0.9.3**: login (incl. MFA),
  teleports, region crossing, chat/IM, inventory, sounds, and a
  15,000-object OAR stress run; the JPEG2000 decoder is bit-exact against
  OpenJPEG. **210 core tests.**
- **Never run against Second Life**, and the Filament view has **never run
  on a physical device**. Those are the Phase 0 gates of the design's
  roadmap and come before feature work: a counts-only live-SL smoke test
  (needs a designated test account) and a physical-device Filament run
  with frame-time numbers.

## Layout

```
core/       pure JVM library: protocol, decoders, geometry, session state
app/        Android app: Compose UI, Filament renderer, foreground service
mockgrid/   runnable fake grid for development (also used by the tests)
materials/  Filament material sources (.mat)
tools/      OpenJPEG fixture generator, environment-test reference script
docs/       provenance, the new-version design, the imported README
```

The README that shipped with the imported tree is preserved verbatim at
`docs/android-kotlin-README.md` — it is the detailed build/test/status
reference (paths inside it refer to the old `android-kotlin/` location).

## Sibling projects

- `Kaleaon/linkpoint-design` — design authority (layout packs, Kthemes)
- `Kaleaon/React-Linkpoint` — the React/TypeScript web companion and the
  origin of this repo's trunk
- `Kaleaon/Linkpoint-Apple` — Linkpoint for Apple platforms (founding repo)
- `Kaleaon/Linkpoint` — the earlier umbrella repo (Rust/Tauri overhaul,
  archived direction; its boundary patterns were adopted, its core was not)

## Licence

MIT — see `LICENSE`. Second Life is a trademark of Linden Lab; this
project is not affiliated with or endorsed by Linden Lab.
