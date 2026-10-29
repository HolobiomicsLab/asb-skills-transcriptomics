---
name: pseudotime-cell-type-correlation-analysis
description: Use when scRNA-seq data has a known biological ordering of cell types
  along a lineage and you need to score pseudotime from Cytopath or baselines (Slingshot,
  Monocle3, PAGA, VeTra, Cellpath) against that ground truth.
when_to_use_negative:
- Root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend trajectory inference with Cytopath in that setting
- The underlying RNA velocity estimation itself is faulty (e.g., spurious root/terminal
  inference as seen on the erythroid gastrulation dataset), making correlated pseudotime
  meaningless
- No independent ground-truth ordering of cell types is available to correlate against
  — the skill depends on known cell type ordering per lineage
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Simulation-based trajectory inference from RNA velocity; supplies pseudotime
    per trajectory over 10 initialisations
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Loads the hippocampus dataset with pre-computed RNA velocity; also source
    of latent time and the velocity-pseudotime directionality for PAGA
- name: Slingshot
  role: Baseline trajectory inference method benchmarked under identical root/terminal
    states
- name: PAGA
  role: Baseline graph abstraction with velocity-pseudotime-based directionality;
    known to produce reversed directionality as a failure mode
- name: Monocle3
  role: Baseline trajectory method; known to fail on disconnected graphs in this benchmark
- name: VeTra
  role: RNA velocity-aware baseline trajectory method
- name: Cellpath
  role: RNA velocity-aware baseline trajectory method
- name: scipy.stats.spearmanr
  role: Computes the Spearman rank correlation between each method's pseudotime and
    known cell type ordering per lineage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:8a04d589407a8a26aa39930e14823c7026769222b3f4f8076fa5940bc280744d
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
  - scipy.stats.spearmanr
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pseudotime-cell-type-correlation-analysis

## Summary

Quantifies whether inferred pseudotime (e.g., Cytopath trajectories on RNA velocity data) recapitulates the known ordering of cell types, by computing per-lineage Spearman correlations and assessing robustness across repeated runs against baseline trajectory methods.

## When to use

Use when scRNA-seq data has a known biological ordering of cell types along a lineage and you need to score pseudotime from Cytopath or baselines (Slingshot, Monocle3, PAGA, VeTra, Cellpath) against that ground truth.

## When NOT to use

- Root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend trajectory inference with Cytopath in that setting
- The underlying RNA velocity estimation itself is faulty (e.g., spurious root/terminal inference as seen on the erythroid gastrulation dataset), making correlated pseudotime meaningless
- No independent ground-truth ordering of cell types is available to correlate against — the skill depends on known cell type ordering per lineage

## Inputs

- AnnData/scvelo object with pre-computed RNA velocity and spliced/unspliced counts (e.g., 18,140-cell developing mouse hippocampus dataset)
- Supplied root and terminal cell states shared across all methods
- Known ordering (rank) of cell types per lineage as ground truth
- Pseudotime vectors per trajectory from each method (Cytopath + Slingshot, PAGA, Monocle3, VeTra, Cellpath)

## Outputs

- Per-method, per-lineage, per-run Spearman correlation table written to hippocampus_pseudotime_correlations.csv
- Robustness assessment of Cytopath correlations across 10 independent initialisations
- Recorded baseline failure modes (e.g., Monocle3 disconnected graph, reversed PAGA directionality)

## How to apply

Load the dataset — here the developing mouse hippocampus dataset (18,140 cells) with pre-computed RNA velocity and supplied root and terminal cell states — via scvelo/anndata. Run Cytopath over 10 independent initialisations under identical root/terminal supervision, inferring pseudotime for each trajectory; Cytopath can infer trajectories with or without root/terminal state supervision. Run the five baselines (Slingshot, PAGA with velocity-pseudotime-based directionality, Monocle3, VeTra, Cellpath) with the identical root and terminal states so correlations are comparable. For each method, lineage, and run, compute the Spearman correlation of inferred pseudotime with the known ordering of cell types using scipy.stats.spearmanr. Assess robustness of Cytopath correlations across the 10 initialisations and record baseline failure modes such as Monocle3's disconnected-graph failure and reversed PAGA directionality, then write the results table to hippocampus_pseudotime_correlations.csv.

## Related tools

- **Cytopath** (Simulation-based trajectory inference from RNA velocity; supplies pseudotime per trajectory over 10 initialisations) — https://github.com/aron0093/cytopath
- **scvelo** (Loads the hippocampus dataset with pre-computed RNA velocity; also source of latent time and the velocity-pseudotime directionality for PAGA)
- **Slingshot** (Baseline trajectory inference method benchmarked under identical root/terminal states)
- **PAGA** (Baseline graph abstraction with velocity-pseudotime-based directionality; known to produce reversed directionality as a failure mode)
- **Monocle3** (Baseline trajectory method; known to fail on disconnected graphs in this benchmark)
- **VeTra** (RNA velocity-aware baseline trajectory method)
- **Cellpath** (RNA velocity-aware baseline trajectory method)
- **scipy.stats.spearmanr** (Computes the Spearman rank correlation between each method's pseudotime and known cell type ordering per lineage)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(pseudotime_per_lineage, cell_type_order)
```

## Evaluation signals

- Spearman correlation between Cytopath pseudotime and known cell type ordering is high for each trajectory and remains consistent across 10 independent initialisations
- All methods are scored against the identical root and terminal cell states on the same 18,140-cell dataset, so correlations are directly comparable
- Correlations are computed per lineage with scipy.stats.spearmanr, producing a complete method × lineage × run table
- Known baseline failure modes are reproduced and recorded: Monocle3 fails on the disconnected graph and PAGA yields reversed directionality
- Output CSV (hippocampus_pseudotime_correlations.csv) contains one correlation value per method, lineage, and run

## Limitations

- Trajectory inference with Cytopath is not meaningful when the underlying RNA velocity estimation itself is faulty (e.g., spurious roots on the erythroid gastrulation dataset)
- Cytopath should not be used when root and terminal cell states are not clearly identifiable
- Robustness of Cytopath pseudotime to root/terminal probability thresholds was only reported as 'data not shown' for the range [0.85, 1]

## Evidence

- [results] high and robust hippocampus correlation: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [results] hippocampus dataset definition: "we applied Cytopath and baseline methods to the developing mouse hippocampus dataset, which was first used to demonstrate RNA velocity of single cells. This dataset is composed of 18,140 cells."
- [readme] root/terminal supervision flexibility: "Cytopath can infer trajectories with or without root/terminal state supervision."
- [discussion] limitation: unclear root/terminal states: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
