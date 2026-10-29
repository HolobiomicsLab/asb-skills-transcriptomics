---
name: cross-run-pseudotime-robustness-assessment
description: 'Use when pseudotime/trajectory output from RNA velocity-based methods
  must be validated: repeated runs of a stochastic method (e.g., 10 Cytopath initialisations)
  need robustness checks and known cell type orderings are available for correlation.'
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty — the authors state
  trajectory inference with Cytopath is not meaningful in that case (e.g., the erythroid
  gastrulation dataset with spurious root/terminal states).
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath in this situation.
- When there is no known cell type ordering or equivalent ground truth available to
  define the Spearman correlation target.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: Stochastic trajectory-inference method run over 10 independent initialisations
    on RNA velocity data to produce pseudotime per trajectory under root/terminal
    supervision
  repo: https://github.com/aron0093/cytopath
- name: Slingshot
  role: Baseline tree-like-topology trajectory inference method run with identical
    root/terminal states for correlation comparison
- name: PAGA
  role: Baseline graph-model method with directionality enabled by velocity pseudotime;
    failure mode of reversed directionality is recorded
- name: Monocle3
  role: Baseline method whose disconnected-graph failure mode is documented in the
    comparison
- name: VeTra
  role: RNA velocity-aware baseline trajectory inference method included in the per-method
    correlation benchmark
- name: Cellpath
  role: RNA velocity-aware baseline trajectory inference method included in the per-method
    correlation benchmark
- name: scvelo
  role: Loading/anndata handling of the hippocampus dataset with pre-computed RNA
    velocity and root/terminal states
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:ad7d5e895f441c686e450cbe054bb3acbc4b1ec4e70e75421afa81ea0a996ea9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cellpath
  - Cytopath
  - Monocle3
  - PAGA
  - Slingshot
  - VeTra
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cross-run-pseudotime-robustness-assessment

## Summary

Assess whether trajectory-inference pseudotime (e.g., Cytopath on RNA velocity fields) is biologically consistent and stable across independent initialisations by computing per-run, per-lineage Spearman correlations against known cell type ordering and benchmarking against baseline methods.

## When to use

Use when pseudotime/trajectory output from RNA velocity-based methods must be validated: repeated runs of a stochastic method (e.g., 10 Cytopath initialisations) need robustness checks and known cell type orderings are available for correlation.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty — the authors state trajectory inference with Cytopath is not meaningful in that case (e.g., the erythroid gastrulation dataset with spurious root/terminal states).
- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in this situation.
- When there is no known cell type ordering or equivalent ground truth available to define the Spearman correlation target.

## Inputs

- anndata/scvelo object with pre-computed RNA velocity (e.g., developing mouse hippocampus dataset, 18,140 cells)
- supervised root and terminal cell states (or clusters defining them)
- known ordering of cell types per lineage (ground-truth ranking for correlation)

## Outputs

- per-method, per-lineage, per-run Spearman correlation table (e.g., hippocampus_pseudotime_correlations.csv)
- robustness assessment of Cytopath correlations across the 10 independent initialisations
- recorded baseline failure modes (e.g., Monocle3 disconnected-graph failure, reversed PAGA directionality)

## How to apply

Load the single-cell dataset (e.g., the developing mouse hippocampus dataset of 18,140 cells with pre-computed RNA velocity in scvelo/anndata format) together with supplied root and terminal cell states. Run Cytopath over multiple independent initialisations (10 in the reference benchmark) under the same root/terminal supervision, and run baselines — Slingshot, PAGA with velocity-pseudotime-based directionality, Monocle3, VeTra, and Cellpath — with identical root/terminal states. For each method, trajectory, and run, compute the Spearman correlation of inferred pseudotime with the known ordering of cell types per lineage using scipy.stats.spearmanr. Judge success by high, stable correlations across runs for the method under test, and record baseline failure modes such as Monocle3's disconnected-graph failure and reversed PAGA directionality. Write the per-method, per-lineage, per-run table (e.g., hippocampus_pseudotime_correlations.csv) for downstream robustness inspection.

## Related tools

- **Cytopath** (Stochastic trajectory-inference method run over 10 independent initialisations on RNA velocity data to produce pseudotime per trajectory under root/terminal supervision) — https://github.com/aron0093/cytopath
- **Slingshot** (Baseline tree-like-topology trajectory inference method run with identical root/terminal states for correlation comparison)
- **PAGA** (Baseline graph-model method with directionality enabled by velocity pseudotime; failure mode of reversed directionality is recorded)
- **Monocle3** (Baseline method whose disconnected-graph failure mode is documented in the comparison)
- **VeTra** (RNA velocity-aware baseline trajectory inference method included in the per-method correlation benchmark)
- **Cellpath** (RNA velocity-aware baseline trajectory inference method included in the per-method correlation benchmark)
- **scvelo** (Loading/anndata handling of the hippocampus dataset with pre-computed RNA velocity and root/terminal states)

## Examples

```
from scipy.stats import spearmanr; rho = spearmanr(cytopath_pseudotime_per_run, known_celltype_ordering_per_lineage)
```

## Evaluation signals

- Spearman correlation between Cytopath pseudotime and known cell type ordering per lineage is high and robust across the 10 independent runs (as in Figure 2D/2E of the reference study).
- All methods were run with identical root and terminal states, so per-method correlation differences are attributable to the inference method, not supervision differences.
- Baseline failure modes are explicitly characterised rather than silently missing (Monocle3 disconnected-graph failure; reversed PAGA directionality).
- The output CSV contains one correlation row per method x lineage x run combination with no missing entries for completed runs.
- Cytopath pseudotime is reported robust to root and terminal probability thresholds within a range of [0.85, 1].

## Limitations

- Robustness of Cytopath pseudotime to root and terminal probability thresholds was reported only as 'data not shown' for the range [0.85, 1].
- Simplifying assumptions of RNA velocity estimation (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene) can make velocity unrepresentative of true dynamics, invalidating the pseudotime assessment.
- Incorrect RNA velocity estimation leads to spurious root/terminal inference (e.g., erythroid gastrulation dataset, fuzzy root-cell probabilities in hepatocyte zonation).

## Evidence

- [results] High, robust per-lineage Spearman correlation across runs: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [results] Hippocampus benchmark dataset size and provenance: "we applied Cytopath and baseline methods to the developing mouse hippocampus dataset, which was first used to demonstrate RNA velocity of single cells. This dataset is composed of 18,140 cells."
- [discussion] Root/terminal threshold robustness range: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [discussion] Supervision requirement caveat: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
- [readme] Cytopath supervision modes (README): "Cytopath can infer trajectories with or without root/terminal state supervision."
