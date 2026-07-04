---
# Machine-readable project descriptor — schema v1 (2026-05-05).
name: py-kids-vc
kind: [cli, library]
status: archived                           # retired 2026-07-04 — superseded by v-pkg (~/vista-forge/v-pkg)
languages: [python]

runtime:
  needs: [python>=3.9]
  optional: []
  excludes: []

distribution:
  pypi: kids-vc                            # ready; not yet published
  github: rafael5/py-kids-vc               # public, pushed 2026-05-05

location: ~/projects/py-kids-vc

exposes:
  cli:
    - kids-vc                              # parse, decompose, assemble, roundtrip, canonicalize
    - kids-vc-merge                        # git merge driver for *.zwr files
  python_api:
    - parse_kid
    - decompose_build
    - assemble_build
    - roundtrip
    - canonicalize_iens
    - canonicalize_routine_line2
  formats_produced:
    - "decomposed KIDComponents/ tree (per-component .m + .zwr)"
    - "*.KID (round-trip / re-assemble)"
  fixtures: src/kids_vc/fixtures/*.kid     # 5 representative .KID samples

consumes:
  formats: ["*.KID", "*.zwr (3-way merge)"]
  services: []

companions:
  - project: py-kids-install
    relation: "downstream — installs the .KID files kids-vc assembles"
  - project: vista-meta
    relation: "vista-meta's `make patch-*` shells out to the kids-vc CLI"
  - project: VistA-DataLoader-fork
    relation: "decompose VISTA_DATALOADER_3P1.KID for git-tracking"

incompatibilities:
  - "Pure-Python; no live VistA needed. But that means it cannot install/verify — pair with py-kids-install for that."

docs:
  primary: README.md
  full_guide: docs/kids-vc-guide.md
  background: docs/kids-vc-background-dev.md
  adr: docs/adr/046-kids-vc-undo-pre-install-snapshot.md
---

# py-kids-vc

> **ARCHIVED 2026-07-04 — superseded by [v-pkg](https://github.com/vista-forge/v-pkg).**
> The Go port carries every capability here (parse/decompose/assemble/
> roundtrip/canonicalize, DD-code extraction, and the ZWR merge driver,
> ported 2026-07-04) and was validated byte-identical on the WorldVistA
> corpus. This repo is kept read-only as the validated reference oracle;
> its design history was imported to the vista-forge docs repo as
> `background/kids-vc-background-dev.md`. Do not develop here.

Version-control tool for VistA KIDS distribution files. Decompose `.KID`
patches into per-component trees for git-tracking; reassemble back to
`.KID` for deployment. Plus a 3-way merge driver for `.zwr` globals.

Full README at [README.md](README.md). Full pipeline reference at
[docs/kids-vc-guide.md](docs/kids-vc-guide.md). Extracted from vista-meta
in May 2026 — see [docs/kids-vc-background-dev.md](docs/kids-vc-background-dev.md).

## Validation

- 5/5 fixture round-trips byte-semantic
- 7/7 ZWR merge tests
- 6/6 XPDK2VC behavioral-contract tests
- 100% round-trip on 2,406 real WorldVistA `.KID` patches (corpus harness in `scripts/`)

## Layout

- `src/kids_vc/` — package: `_impl.py` (canonical 1053-line implementation), `merge.py`, `cli.py`, `fixtures/`
- `tests/` — standalone test scripts (run via `python tests/test_*.py`, not pytest)
- `scripts/fetch_kids_corpus.py` — corpus harness
- `docs/` — guide, background, ADR-046
- `.github/workflows/ci.yml` — round-trip + merge + compat + lint
