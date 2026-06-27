# Reuse Manifest

This repository was copied from `/home/gpus/reading-not-reasoning` on 2026-06-27 with no original Git history.

## Included Because They Directly Support The New Research

| Path | Reason |
|---|---|
| `docs/numtrace_spec.md` | New research specification. |
| `todo/0628.md` | New research TODO plan. |
| `todo/06.md` | Prior in-place KV intervention design and target-selection audit. |
| `todo/0627.md` | Review guardrails reused as paper-readiness gates. |
| `scripts/build_n1_curriculum.py` | FinQA oracle, executable programs, flippable operands, table rendering. |
| `scripts/battery_n1_targeted.py` | FinQA targeted corruption and propagation helpers. |
| `scripts/build_tabmwp_test.py` | TabMWP free-numeric data build path. |
| `scripts/battery_n400.py` | Legacy diagnostic battery reference; not headline evidence. |
| `scripts/probe_n400.py` | Legacy diagnostic probe reference; not headline evidence. |
| `app/distill/` | Shared grading, statistics, JSON, and result-store helpers. |
| `app/config.py`, `app/vqa.py`, `app/eval_fingerprint.py` | Small dependencies required by copied helpers. |
| `docs/paper/references.bib` | Seed bibliography for related work expansion. |
| `tests/` | Baseline tests for copied grading/statistics helpers. |

## Explicitly Excluded

- `.git/` from the source repo.
- `data/` runtime contents.
- `logs/`.
- model weights and adapter artifacts.
- old paper PDF and LaTeX build byproducts.
- old broad experiment scripts not needed for FinQA/TabMWP/path-specific audit.

## Evidence Boundary

The old random two-pass battery is preserved only for provenance and comparison. New paper claims should use new artifacts generated under:

- `twopass_onpath`
- `inplace_text`
- `path_specific_visual_text`

Any use of `twopass_random` must be labelled as diagnostic/background.
