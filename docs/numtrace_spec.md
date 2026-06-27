# NumTrace: Tracing Numerical Answer Paths in VLMs

日期：2026-06-28
状态：new research line spec v0
对应执行计划：`todo/0628.md`
继承项目：`Reading, Not Reasoning`
目标 venue：ICLR 2027 方向；AAAI 2027 当前稿仅复用其中成熟的 scope-safe 部分。

---

## 1. Research Thesis

**Core thesis**

Numerical VLM answers may be produced through multiple partially independent paths: a localized visual numeric source, an emitted textualized numeric intermediate, and latent hidden-state computation. Existing CoT corruption probes mostly intervene on written rationales. This project estimates path-specific causal effects by independently intervening on localized visual sources and emitted numeric tokens inside the same autoregressive stream.

**Primary research question**

In chart/table numerical tasks, does the final answer causally depend on:

1. the localized visual numeric source,
2. the textualized numeric intermediate emitted in the CoT,
3. both paths through conflict arbitration,
4. or neither observable path, implying recovery through another latent route?

**Short name**

`NumTrace`

**Candidate title**

`NumTrace: Tracing Numerical Answer Paths in Vision-Language Models`

---

## 2. Relation to the Current Paper

This is a new research line built on the infrastructure and empirical lessons of `Reading, Not Reasoning`, not merely an incremental repair of that paper.

The current paper asks:

> Does CoT SFT make the emitted written chain load-bearing for final answers?

This new project asks:

> Given a particular numerical answer path, how much causal influence flows through the visual source, the emitted numeric token, and their conflict-resolution behavior?

The old project contributes motivating evidence and reusable machinery, but its strongest claims must not be silently transferred. In particular:

- Old `twopass_random` battery results are diagnostic background, not clean on-path path-specific evidence.
- FinQA gold-program targeted logic is reusable as a clean oracle substrate.
- In-place KV intervention is part of the new method, but by itself estimates only `textualized number -> answer`; it does not prove visual grounding.
- Full visual-source claims require source localization, counterfactual rendering/editing, source-level answer recomputation, and a conflict design.

---

## 3. Key Constructs

**Visual source**

A localized numeric source in the input image or rendered table/chart: a table cell, chart mark, line point, bar height, region+legend value, object set, or diagram relation.

**Textualized numeric intermediate**

A number emitted by the model in its CoT that corresponds to, copies, computes from, or otherwise mediates the selected visual source.

**Final numeric answer**

The answer extracted and graded under task-specific numeric normalization.

**Counterfactual visual source**

An edited or re-rendered input where only the selected source is changed from `v_true` to `v_cf`, while non-target evidence and task format are held fixed as far as possible.

**Text intervention**

An in-place generation branch that forces the textualized numeric token/span from `v_true` to `v_cf` using the original prefix cache. No second prompt and no post-hoc continuation instruction are allowed for the C1-clean estimate.

**Not audited**

All hidden-state computation paths not mediated by the selected source/token path. The project may detect their presence indirectly, but does not claim to fully identify them.

---

## 4. Causal Design

Each valid target is converted into a 2x2 path intervention:

| visual source | emitted / forced CoT number | cell | purpose |
|---|---|---|---|
| true | true | `V_true_T_true` | baseline sanity |
| false | false | `V_cf_T_cf` | counterfactual coherence check |
| false | true | `V_cf_T_true` | tests visual source influence with text held fixed |
| true | false | `V_true_T_cf` | tests textualized-number influence with visual source held fixed |

For every target:

- `v_true`: original source value and original CoT value.
- `v_cf`: source-level counterfactual value.
- `c_true`: answer under original source.
- `c_cf`: answer recomputed after replacing the selected source with `v_cf`.
- `T_cf`: forced inside the original autoregressive stream when possible.
- `V_cf`: produced through source-level counterfactual rendering/editing, not prompt text editing.

Primary effects:

- `E_visual_to_text = P(CoT_number = v_cf | V_cf) - P(CoT_number = v_cf | V_true)`
- `E_text_to_answer = P(answer = c_cf | V_true, T_cf) - P(answer = c_cf | V_true, T_true)`
- `E_visual_direct_to_answer = P(answer = c_cf | V_cf, T_true) - P(answer = c_cf | V_true, T_true)`
- `conflict_winner`: `visual_cf`, `text_cf`, `snap_true`, `both_cf`, or `other`

---

## 5. Hypotheses and Interpretable Outcomes

**H1: Visual-source-dominant path**

`E_visual_to_text` and/or `E_visual_direct_to_answer` is high, while `E_text_to_answer` is low. Interpretation: the emitted written number is not the main load-bearing route; the model recovers from visible evidence or a latent source-conditioned state.

**H2: Text-mediated path**

`E_text_to_answer` is high and `V_true_T_cf` follows `c_cf`. Interpretation: the textualized numeric intermediate is load-bearing within the native stream.

**H3: Mixed mediation / arbitration**

Both visual and text interventions move the answer, and conflict cells split. Interpretation: answer formation uses both source evidence and emitted text, with dataset- or model-specific arbitration.

**H4: Neither observable path dominates**

Both effects are low or unstable. Interpretation: the selected source/token path is incomplete, the counterfactual is out of distribution, or the model uses another latent route. This is a valid outcome, not a failed experiment.

**Divergence rule**

If in-place follow is much higher than two-pass follow, the result changes the thesis: prompt-level re-query may destroy stream faithfulness. The paper must then avoid saying written chains are not load-bearing in general.

---

## 6. Data Strategy

Datasets are selected by path-identification feasibility, not benchmark prestige.

Five selection criteria:

1. `source_localizable`: can the relevant cell/mark/region/object set be localized?
2. `source_editable`: can a clean visual counterfactual be rendered or edited?
3. `answer_recomputable`: can `c_cf` be computed after changing the source?
4. `cot_token_locatable`: does the model emit a numeric intermediate that can be matched?
5. `visual_realism`: does the substrate carry external validity?

Planned tiers:

| tier | substrate | source family | role |
|---|---|---|---|
| v1 | FinQA rendered table | table cell | clean gold-program proof of concept |
| v1 | synthetic rendered table | table cell | renderer and intervention spike |
| v1.5 | TabMWP | table cell / expression operand | extension to current headline table arithmetic |
| v2 | PlotQA / DVQA / FigureQA / LEAF-QA | chart mark / data point | first non-table visual numeric source |
| v2 | TAT-QA / HiTab / MultiHiertt | realistic table cell | external validity in financial tables |
| v3 | counting, maps, diagrams, scientific figures | object set / region / relation | source-family generalization |

Generalized claims require at least two source families, for example table cell plus chart mark.

---

## 7. Experimental Units and Eligibility

Unit of analysis:

One `sample_id + target_id` pair, where `target_id` binds the visual source, textualized numeric token/span, `v_true`, `v_cf`, `c_true`, and `c_cf`.

Eligibility requirements:

- base generation is correct under the task grader,
- a CoT or rationale is present,
- source is unique and editable,
- `c_cf` can be recomputed,
- target numeric span is uniquely locatable before the final answer,
- for v1, both `v_true` and `v_cf` are single-token surfaces,
- `c_cf != c_true`,
- conflict cells are identifiable; if source- and text-implied answers collapse to the same value, conflict arbitration is skipped.

Skipped samples remain in JSONL with explicit `skip_reason`.

---

## 8. Methods

### 8.1 On-Path Locator

The locator maps model output and dataset metadata to an `OnPathTarget`:

`sample_id, target_id, t_star/token_span, v_true, v_cf, c_true, c_cf, single_token, backend, evidence`

Backends:

- FinQA: gold program and flippable table operands.
- TabMWP: conservative safe arithmetic parser.
- Chart family: high-precision chart metadata or parseable expression subset.

### 8.2 Visual Source Backend

The source backend returns a `VisualSourceTarget`:

`sample_id, source_id, source_type, source_region, source_surface, v_true, v_cf, c_true, c_cf, editable, renderer_backend, evidence`

It must also generate or reference `V_true` and `V_cf` artifacts.

### 8.3 In-Place KV Intervention

The C1-clean text intervention uses HuggingFace manual greedy decode:

1. prefill the original multimodal prompt,
2. decode token-by-token with cache,
3. locate the target span,
4. clone/crop the prefix cache before the span,
5. force `v_cf` token(s),
6. continue greedy decoding without a new prompt.

Replay sanity is required: forcing the original `v_true` span from the same prefix should preserve the final answer.

### 8.4 Same-Target Two-Pass Baseline

Two-pass on-path runs remain useful as a comparison, but they must use the same located targets and `c_cf`, not the old random number selector.

### 8.5 Controls

Minimum controls:

- `replay_true`: cache sanity only.
- `local_filler`: local no-target disruption.
- `offpath_number`: non-dependent numeric replacement when available.

Optional v2 controls:

- forced filler suffix,
- forced shuffle suffix,
- deletion,
- paraphrase with numeric multiset guard.

Controls do not report `follow` unless they inject a defined counterfactual answer target.

---

## 9. Metrics and Statistical Plan

Text-path readout:

- `snap`: answer matches `c_true`.
- `follow`: answer matches `c_cf`.
- `other`: neither.
- `flip`: final answer differs from baseline answer.

Conflict readout:

- `both_cf`,
- `visual_cf`,
- `text_cf`,
- `snap_true`,
- `other`,
- `non_identifiable_conflict`.

Statistical reporting:

- follow/snap/flip rates with 95% binomial CIs,
- paired bootstrap or exact randomization CI for `corrupt_flip - control_flip`,
- paired bootstrap CI for path effects where all required cells are present,
- raw-to-valid selection flow,
- source-flow table,
- sensitivity bounds for failed locator/source cases,
- confirmatory vs diagnostic labels for multi-cell comparisons.

---

## 10. Reuse Plan

Reusable from the existing repo:

- `scripts/build_n1_curriculum.py`: FinQA program parser/executor, flippable operand logic, table-to-text and table rendering helpers.
- `scripts/battery_n1_targeted.py`: FinQA targeted corruption, consistent propagation, answer extraction, relaxed numeric grading patterns.
- `app/distill/eval_common.py`: relaxed numeric/text grading primitives, candidate parsing where relevant.
- `app/distill/common.py`: JSON/JSONL helpers and stable hashes.
- `app/distill/seed_runner.py`: append-only result-store pattern.
- `scripts/battery_n400.py` and `scripts/probe_n400.py`: prompt conventions, task loading, old diagnostic baseline only.
- `scripts/serve/`: local model serving commands for old vLLM-based experiments; in-place KV itself must use HuggingFace.
- `docs/snapshots/`: milestone record style.
- `docs/paper/references.bib`: related-work seed bibliography, subject to verification and expansion.

Must be newly implemented:

- dataset-independent target/source schemas,
- source counterfactual renderers,
- HuggingFace manual decode and cache-branch intervention,
- same-target on-path two-pass baseline,
- path-specific four-cell runner,
- source-flow and path-effect summarizers,
- tests for locators, renderers, readouts, cache replay, and statistics.

Must not be reused as headline evidence:

- old generic `corrupt_number` as on-path evidence,
- old `follow == injected number` readout when `c_follow` is not recomputed,
- old two-pass random ChartQA/TabMWP numbers as C1/C2-clean causal estimates,
- mechanism language such as "visual re-read" without visual counterfactual evidence.

---

## 11. Artifact Contract

All new outputs are JSONL, one row per attempt or path cell.

Required common fields:

`project, paradigm, dataset, source_family, condition, model_id, checkpoint_id, sample_id, target_id, intervention, skipped, skip_reason, seed, code_fingerprint`

Text-path fields:

`token_span, v_true, v_cf, c_true, c_cf, base_ans, inj_ans, flip, readout`

Source/path fields:

`path_cell, source_id, source_type, source_region, source_surface, source_true, source_cf, c_source_true, c_source_cf, text_forced, answer_source_readout, answer_text_readout, conflict_winner`

Reproducibility fields:

`runner, runner_args, git_sha, dirty, input_artifact_hash, renderer_hash, prompt_hash, decode_config, tokenizer_surface, save_text_policy`

Summaries must be generated from JSONL only. No paper number may be hand-entered.

---

## 12. Paper Strategy

**AAAI 2027 fallback/current-scope paper**

Use the existing `Reading, Not Reasoning` scope. Incorporate only mature guardrails: claim boundary, statistics, selection flow, cleaner controls if ready, and possibly a small clean FinQA appendix. Do not promise the full path-specific audit.

**ICLR 2027 target paper**

Make this project the main contribution. The minimum credible ICLR version should include:

- FinQA table-cell path-specific four-cell results,
- TabMWP table-cell/text-path extension,
- one non-table source family or a clearly justified high-precision chart family,
- same-target two-pass vs in-place comparison,
- full source-flow and selection-flow accounting,
- related-work positioning against visual dependence, grounded CoT, saliency/attribution, and rationale faithfulness work.

**Non-overlap discipline**

If both submissions overlap in time, the ICLR version must have a new research question, new method, new primary experiments, and new conclusions. The AAAI paper can be cited or described as motivating prior work only after venue policy permits.

---

## 13. Kill Criteria and Scope Controls

Stop or downscope if:

- HuggingFace cache replay cannot reach reliable sanity,
- valid n is too low after multi-token support,
- source counterfactuals are not read by the model in coherence cells,
- source edits introduce obvious artifacts,
- path effects are driven by a small unrepresentative subset,
- the result cannot be reproduced from JSONL and runner fingerprints.

Acceptable downscopes:

- FinQA-only method note,
- table-cell-only paper,
- text-path-only in-place paper,
- diagnostic appendix rather than headline result for a difficult source family.

The project should follow the evidence rather than preserve the original `Reading, Not Reasoning` title.
