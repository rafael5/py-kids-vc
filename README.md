# kids-vc

Version-control tool for **VistA KIDS** (Kernel Installation and Distribution System) distribution files. Decompose `.KID` patches into per-component files for git-tracking; reassemble back to `.KID` for deployment.

## What it does

KIDS bundles routines, FileMan data-dictionary changes, options, protocols, RPCs, and install logic into one monolithic `.KID` text file. Git's line-based diff/merge is destructive for this format because adjacent entries are semantically independent.

`kids-vc` provides:

- **`decompose`** — `.KID` → per-component directory tree (routines as `.m`, FileMan DDs as `.zwr`, Kernel components per-entry)
- **`assemble`** — directory tree → `.KID` (reversible)
- **`roundtrip`** — verify lossless round-trip
- **`canonicalize`** — substitute install-time IENs with stable `"IEN"` placeholder for cross-instance diff stability
- **`parse`** — summarize a `.KID` without decomposing

Plus `kids-vc-merge` — a git merge driver that does entry-level 3-way merge on `.zwr` files (`.gitattributes`-installable).

## Install

```bash
# From source (editable)
pip install -e .

# From PyPI (when published)
pip install kids-vc
```

This installs two console scripts: `kids-vc` and `kids-vc-merge`.

## Usage

```bash
# Parse summary
kids-vc parse OR_3.0_484.KID

# Decompose for git tracking
kids-vc decompose OR_3.0_484.KID ./patches/

# Reassemble
kids-vc assemble ./patches/ rebuilt.KID

# Verify round-trip
kids-vc roundtrip OR_3.0_484.KID

# Canonicalize IENs (lossy — for cross-instance diffs)
kids-vc canonicalize ./patches/
```

### git merge driver

```bash
echo '*.zwr merge=zwr' >> .gitattributes
git config merge.zwr.name "ZWR entry-level merge"
git config merge.zwr.driver "kids-vc-merge %O %A %B"
```

## Background

Two prior VistA-on-git bridge attempts:

- **SKIDS** (WorldVistA, 2011–2013) — three disjoint spikes (Python `ParseKIDS.py` + MUMPS `ZDIOUT1.m` + unmerged Java `KIDSAssembler`) that never integrated. Abandoned.
- **XPDK2VC** (Sam Habiel, 2014–2020) — coherent in-VistA MUMPS implementation, 4 routines (~870 lines), handled 13 KIDS component types with diff-stability engineering.

`kids-vc` is a Python port of XPDK2VC's architecture, plus features neither predecessor had:

- **DD-embedded MUMPS extraction** — input transforms, xref SET/KILL, computed-field code extracted as per-field `.m` annotations
- **ZWR 3-way merge driver** — git-native entry-level merges for `.zwr` files
- **Corpus-verified** — 100% round-trip pass rate on all 2,406 real WorldVistA `.KID` patches (~3.5M subscripts)

## Documentation

| Topic | File |
|---|---|
| **Full user guide** — workflows, patch lifecycle, internals | [docs/kids-vc-guide.md](docs/kids-vc-guide.md) |
| Design history, prior art, decisions | [docs/kids-vc-background-dev.md](docs/kids-vc-background-dev.md) |
| ADR-046 — pre-install snapshot for undo | [docs/adr/046-kids-vc-undo-pre-install-snapshot.md](docs/adr/046-kids-vc-undo-pre-install-snapshot.md) |

## Layout

```
src/kids_vc/        # package
  _impl.py          # parse/decompose/assemble/canonicalize/roundtrip
  merge.py          # ZWR 3-way merge driver
  cli.py            # console script entry point
  fixtures/*.kid    # test fixtures (DG, OR, VMDD, VMTEST, XU)
tests/              # standalone test scripts
  test_zwr_merge.py        # merge driver — 7 cases
  test_xpdk2vc_compat.py   # XPDK2VC behavioral-contract checks
scripts/
  fetch_kids_corpus.py     # corpus harness (every .KID in WorldVistA/VistA)
.github/workflows/ci.yml   # roundtrip + merge + compat + lint
```

## Development

```bash
pip install -e .
python3 tests/test_zwr_merge.py
python3 tests/test_xpdk2vc_compat.py

# Round-trip all fixtures
for f in src/kids_vc/fixtures/*.kid; do
  kids-vc roundtrip "$f"
done

# Full corpus run (downloads ~2,400 .KID files, ~10 min)
python3 scripts/fetch_kids_corpus.py
```

## Companion

For runtime install/verify against a live YottaDB VistA instance, see the separate **`py-kids-install`** project. The intended pipeline is:

```
decompose → edit/git → assemble → install → verify
└──── kids-vc ────┘    └─── py-kids-install ───┘
```

## Origin

Extracted from the [vista-meta](https://github.com/rafael5/vista-meta) project (May 2026), where it was developed as Phase 8 of the KIDS-version-control initiative.

## License

Apache 2.0 (matches both SKIDS and XPDK2VC).
