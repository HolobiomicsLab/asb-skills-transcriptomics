---
name: scaffold-simulation-dataset-generation
description: Use when you need simulated scRNA-seq benchmarks with known ground truth
  — e.g., testing whether a trajectory-suitability check correctly flags trajectory-free
  simulations (homogeneous cells) while not falsely flagging linear-trajectory simulations.
when_to_use_negative:
- Real experimental datasets with unknown ground truth — simulation is only needed
  when trajectory truth must be known to validate flag accuracy; for real data use
  Escort directly on the dataset (e.g., GSE190616).
- Benchmarks requiring non-linear (bifurcating/tree) trajectory topologies — the documented
  simulation conditions here cover only homogeneous (0% dynamic) and linear (20% dynamic
  genes) trajectories.
- If you only need ready-made benchmarks — the article notes six publicly available
  simulated datasets [45] that can be used instead of generating new ones.
tools:
- name: Scaffold
  role: R package (v0.2.0) used with estimateScaffoldParameters() and simulateScaffold()
    to generate simulated scRNA-seq datasets from the human pancreas reference
- name: Escort
  role: R package used to load each simulated dataset and run Step 1 checks (HD_DCClusterscheck,
    step1_testHomogeneous) to produce suitability flags
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Runtime environment in which Scaffold simulation and Escort Step 1 checks
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:7985fdaae708a2b32a12539b0d2222664ebdf87a5b83b00347d99cc59d16c9b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - R
  - Scaffold
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scaffold-simulation-dataset-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate synthetic scRNA-seq datasets with the Scaffold R package (v0.2.0) from a human pancreas single-cell RNA-seq reference, spanning known ground-truth conditions (homogeneous, 0% dynamic genes vs. linear trajectory, 20% dynamic genes) so that downstream tools like Escort's Step 1 suitability checks can be validated against known truth.

## When to use

Use when you need simulated scRNA-seq benchmarks with known ground truth — e.g., testing whether a trajectory-suitability check correctly flags trajectory-free simulations (homogeneous cells) while not falsely flagging linear-trajectory simulations.

## When NOT to use

- Real experimental datasets with unknown ground truth — simulation is only needed when trajectory truth must be known to validate flag accuracy; for real data use Escort directly on the dataset (e.g., GSE190616).
- Benchmarks requiring non-linear (bifurcating/tree) trajectory topologies — the documented simulation conditions here cover only homogeneous (0% dynamic) and linear (20% dynamic genes) trajectories.
- If you only need ready-made benchmarks — the article notes six publicly available simulated datasets [45] that can be used instead of generating new ones.

## Inputs

- Human pancreas single-cell RNA-seq reference dataset (simulation reference, Dong et al. ref [44])
- Scaffold v0.2.0 configuration produced by estimateScaffoldParameters()
- Simulation condition settings: percent dynamically expressed genes (0% homogeneous vs. 20% linear trajectory)

## Outputs

- Simulated scRNA-seq count datasets (one trajectory-free / homogeneous, one 20%-dynamic-genes linear trajectory)
- Escort Step 1 suitability flags compiled into step1_suitability_results.csv
- Per-dataset flag labels: unsuitable-for-trajectory vs. suitable

## How to apply

In R, install Scaffold v0.2.0 and configure simulation parameters from the human pancreas single-cell RNA-seq reference using estimateScaffoldParameters(), then run simulateScaffold() to produce each synthetic dataset. Generate at least two contrast conditions: one with no dynamically expressed genes (homogeneous cells, no true trajectory) and one with 20% dynamic genes yielding a linear trajectory. Load each simulated dataset with the Escort package and run the Step 1 cluster-disconnectedness check (HD_DCClusterscheck: scLCA clustering, Manhattan distance matrix, within/between-cluster Jaccard scores, two clusters connected only if >10 cells per cluster meet the 0.3 Jaccard cutoff) and the homogeneity test (step1_testHomogeneous: Spearman correlations of PC1 against a random subset of the top 100 highly variable genes, 20,000-iteration permutation test with FDR correction, homogeneity flagged when significant correlations at p < 0.05 exceed 46% of genes). Compile the Step 1 flags into step1_suitability_results.csv for both simulations. The rationale is that only simulations with known trajectory truth can cleanly validate whether suitability flags detect (and do not over-detect) trajectory-free data.

## Related tools

- **Scaffold** (R package (v0.2.0) used with estimateScaffoldParameters() and simulateScaffold() to generate simulated scRNA-seq datasets from the human pancreas reference)
- **Escort** (R package used to load each simulated dataset and run Step 1 checks (HD_DCClusterscheck, step1_testHomogeneous) to produce suitability flags) — https://github.com/xiaorudong/Escort
- **R** (Runtime environment in which Scaffold simulation and Escort Step 1 checks are executed)

## Examples

```
library(Scaffold); params <- estimateScaffoldParameters(pancreasRef); sim_homogeneous <- simulateScaffold(params, percent.dynamic.genes = 0); sim_linear <- simulateScaffold(params, percent.dynamic.genes = 20)
```

## Evaluation signals

- Every simulated dataset with 0% dynamic genes (no true trajectory) is detected and flagged by Escort's Step 1 as unsuitable for trajectory fitting
- The 20%-dynamic-genes linear-trajectory simulation is NOT falsely flagged by Step 1
- Flagged trajectory-free simulations produce even less accurate trajectories than non-recommended embeddings, confirming the flag's predictive value
- Both simulations are generated from the same human pancreas reference via estimateScaffoldParameters()/simulateScaffold(), differing only in the percent dynamic genes setting
- step1_suitability_results.csv contains one flag per simulation and matches the ground-truth condition (flagged for 0% dynamic, unflagged for 20% dynamic)

## Limitations

- Escort (the downstream validator) only considers trajectory inference approaches using two-dimensional representations; methods outside this restriction require further evaluation
- Escort is not guaranteed to identify a recommended embedding among evaluated choices, so simulated datasets with true trajectories may still yield no recommended embedding
- No software version is specified for Escort or R in the article; only Scaffold v0.2.0 is pinned in the workflow

## Evidence
