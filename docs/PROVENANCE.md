# Provenance — the Linkpoint-Android import

This repository was created on **2026-11-25** at Joe's direction ("make a
GOOD repo for all of Linkpoint for Android"), settling the repo-home open
item in the new-version design (`docs/NEW-VERSION-DESIGN.md` §4/§12).

## What was imported

The complete `android-kotlin/` tree from **`Kaleaon/React-Linkpoint`
GitHub main, commit `67ea26df795721a1a3f021843e5b818fda81d751`** (merge of
PR #420, 2026-11-24) — 266 files, ~1.8 MB. The tree's internal structure is
preserved exactly, re-rooted at this repository's root (it is a standalone
Gradle project). Every file's Git blob SHA-1 was verified against the
source tree during the import; the remote tree hash was verified after
the push.

The import was taken from GitHub main, **not** from the local
React-Linkpoint checkout (which was ~7 weeks stale, 2026-10-04).

## Changes made at import (additions and wrapping only)

- `README.md` — new top-level README for this repo. The imported tree's
  own README is preserved **verbatim** at `docs/android-kotlin-README.md`
  (paths inside it refer to the old `android-kotlin/` location).
- `.gitignore` — extended to the standard Android/Kotlin set; every entry
  from the imported `.gitignore` is retained.
- `LICENSE` — added; MIT, byte-identical to the licence carried by
  `Kaleaon/Linkpoint`.
- `docs/NEW-VERSION-DESIGN.md` — copy of the 2026-11-25 new-version
  design study from the Linkpoint design workspace.
- `docs/PROVENANCE.md` — this note.

**No imported source file's content was modified.**

## Authorship

The imported code is Joe's (Kaleaon) work from the React-Linkpoint
project, built up with AI assistance (Stitch/agy sessions) as recorded in
that repo's history, which remains the full development record of the
code up to the import commit.
