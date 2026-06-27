# NumTrace

New research-line repository derived from `Reading, Not Reasoning`, without the original Git history.

This repo focuses on a narrower causal question:

> In chart/table numerical VLM tasks, does the final answer depend on a localized visual numeric source, a textualized CoT number, both, or neither observable path?

Primary spec:

- `docs/numtrace_spec.md`

Execution plan:

- `todo/0628.md`

Legacy/background notes carried over for context:

- `todo/06.md` - in-place KV intervention spec and old target-selection audit.
- `todo/0627.md` - review guardrails for statistics, selection flow, controls, and claim boundaries.
- `docs/snapshots/n1_prereg_0625.md`
- `docs/snapshots/n1_complete_0625.md`

## What Was Copied

Reusable code:

- `scripts/build_n1_curriculum.py` - FinQA program parser/executor, flippable operand logic, table rendering.
- `scripts/battery_n1_targeted.py` - FinQA targeted probe and consistent propagation utilities.
- `scripts/build_tabmwp_test.py` - TabMWP free-numeric data builder.
- `scripts/battery_n400.py` and `scripts/probe_n400.py` - old diagnostic battery reference only.
- `app/distill/` - grading, statistics, JSON helpers, result-store helpers.
- `app/config.py`, `app/vqa.py`, `app/eval_fingerprint.py` - small dependencies needed by copied helpers.
- `scripts/serve/` - local serving helpers for old HTTP-style evaluation.

Not copied:

- Old `.git` history.
- Runtime data under `data/`.
- Logs.
- Model weights / adapters.
- Paper build artifacts.
- The old full paper draft except `docs/paper/references.bib`.

## Evidence Boundary

Old random two-pass ChartQA/TabMWP corruption results are diagnostic background only. New headline evidence must be built with same-target, source-localized, answer-recomputable interventions:

- `twopass_onpath`
- `inplace_text`
- `path_specific_visual_text`

## Setup

The original project used a conda env named `mbe-up`. A minimal equivalent is:

```bash
conda create -n numtrace python=3.11 -y
conda run -n numtrace pip install torch --index-url https://download.pytorch.org/whl/cu124
conda run -n numtrace pip install -r requirements.txt
```

Smoke test:

```bash
pytest
```

## First Development Tasks

1. Write `docs/snapshots/0628_path_prereg.md`.
2. Write `docs/snapshots/0628_dataset_selection.md`.
3. Implement `app/eval_distill/onpath.py` and `app/eval_distill/visual_source.py`.
4. Extract FinQA locator/source helpers from the copied FinQA scripts.
5. Spike HuggingFace Qwen3-VL manual greedy decode and cache replay.
