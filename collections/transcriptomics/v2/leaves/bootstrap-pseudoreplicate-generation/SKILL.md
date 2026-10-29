---
name: bootstrap-pseudoreplicate-generation
description: Use when a single-cell dataset in AnnData (.h5ad) format lacks biological
  replicates — i.e., samples_col is None or one sample per condition — and differential
  cell-type proportion analysis between conditions is still required.
when_to_use_negative:
- The dataset already has true biological replicates for every condition — run the
  standard scanpro(adata, samples_col=...) regression path instead of bootstrapping,
  since bootstrapping is designed for non-replicated data.
- The cell type of interest is a rare cluster (<~2% of all cells) — the paper's benchmarking
  found bootstrapping has limited benefits for rare cell types and such clusters produce
  unstable results.
- The goal is definitive inference on variability rather than an indication — results
  from pseudo-replicates should be interpreted with caution and validated experimentally,
  so this skill is the wrong choice when fully powered replicate-based analysis is
  available.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Scanpro
  role: Implements the iteration-based bootstrapping to simulate pseudo-replicates
    and the propeller-style linear regression (anova/t_test) applied to each pseudo-replicate
    split; invoked via scanpro() or run_scanpro with a chosen number of pseudo-replicates
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Reads the .h5ad input and provides the single-cell framework into which Scanpro
    integrates for proportion analysis
- name: AnnData
  role: Native input object format holding the cell-by-cluster metadata (samples_col,
    clusters_col, conds_col) required by scanpro()
- name: propeller
  role: Supplies the linear regression approach on logit/arcsin-sqrt transformed proportions
    that Scanpro extends with bootstrapping
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:8adbedd97339aed45d20c720382a2b626acc7298efc3ae9b64b7160d05508290
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  - propeller
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# bootstrap-pseudoreplicate-generation

## Summary

Generate pseudo-replicates from unreplicated single-cell datasets by iteratively splitting each sample's cells into n_reps groups via bootstrapping without replacement, enabling robust cell-type proportion analysis with Scanpro when biological replicates are absent.

## When to use

Use when a single-cell dataset in AnnData (.h5ad) format lacks biological replicates — i.e., samples_col is None or one sample per condition — and differential cell-type proportion analysis between conditions is still required.

## When NOT to use

- The dataset already has true biological replicates for every condition — run the standard scanpro(adata, samples_col=...) regression path instead of bootstrapping, since bootstrapping is designed for non-replicated data.
- The cell type of interest is a rare cluster (<~2% of all cells) — the paper's benchmarking found bootstrapping has limited benefits for rare cell types and such clusters produce unstable results.
- The goal is definitive inference on variability rather than an indication — results from pseudo-replicates should be interpreted with caution and validated experimentally, so this skill is the wrong choice when fully powered replicate-based analysis is available.

## Inputs

- AnnData (.h5ad) single-cell object with sample, cluster, and condition metadata columns (e.g., scanpro_tutorial.h5ad)
- n_reps: number of pseudo-replicates to split each sample into (e.g., 2, 3, 4, 5, 8, 10, or 14)
- n_min: minimum cluster cell-count threshold for trimming unstable clusters

## Outputs

- pseudo_replicate_cells.csv: cell-to-pseudo-replicate assignment table covering all input cells per sample
- pooled_statistics.csv: one row per retained cluster with finite pooled beta coefficient (Rubin's rule) and median adjusted p-value
- out.results object from scanpro with per-cluster differential proportion calls

## How to apply

Load the dataset with scanpy/anndata and extract the sample, cluster, and condition metadata columns used by scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'). First apply n_min trimming, dropping clusters supported by fewer than n_min cells (the paper's benchmarking found clusters below ~2% of all cells give unstable results). Then, for each of ~100 iterations, split every sample's cells into n_reps pseudo-replicates by sampling cells without replacement, recording the cell-to-pseudo-replicate assignments. In each iteration, compute per-cluster cell proportions and fit the propeller-style linear regression of proportions on condition (reusing the exported anova/t_test helpers from scanpro.scanpro) to obtain per-cluster beta coefficients and p-values. Pool beta coefficients across iterations with Rubin's multiple-imputation rule and take the median of adjusted p-values per cluster; choose n_reps according to mean total cell count per condition — 2 and 4 pseudo-replicates for up to ~5,000 cells, 8–14 for datasets above ~25,000 cells.

## Related tools

- **Scanpro** (Implements the iteration-based bootstrapping to simulate pseudo-replicates and the propeller-style linear regression (anova/t_test) applied to each pseudo-replicate split; invoked via scanpro() or run_scanpro with a chosen number of pseudo-replicates) — https://github.com/loosolab/scanpro
- **Scanpy** (Reads the .h5ad input and provides the single-cell framework into which Scanpro integrates for proportion analysis)
- **AnnData** (Native input object format holding the cell-by-cluster metadata (samples_col, clusters_col, conds_col) required by scanpro())
- **propeller** (Supplies the linear regression approach on logit/arcsin-sqrt transformed proportions that Scanpro extends with bootstrapping)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, clusters_col='cluster', conds_col='conds', samples_col=None)
```

## Evaluation signals

- pseudo_replicate_cells.csv accounts for every input cell per sample with no cells lost or duplicated across pseudo-replicates
- pooled_statistics.csv contains exactly one row per retained (n_min-trimmed) cluster with a finite pooled beta coefficient and a median adjusted p-value column
- auROC of the bootstrapped calls behaves as reported: good auROC for 2 and 4 pseudo-replicates up to ~5,000 mean cells and for 8–14 pseudo-replicates above ~25,000 cells
- Bootstrapped conclusions match those from truly replicated datasets in comparable benchmark scenarios (the paper reports the approach 'often results in the same conclusion as replicated datasets')
- Sensitivity and false-positive rates computed against simulated ground truth (via sim_scanpro with 100 simulated replicate datasets) fall within expected benchmark ranges

## Limitations

- Bootstrapping has limited benefits when handling rare cell types; clusters supported by less than 2% of all cells produce unstable results
- Bootstrapped results should be seen as an indication of cell types with proportional shifts and interpreted with caution, requiring additional experimental validation
- The lack of replicates limits the ability to draw definitive conclusions about variability in the data and per se is not advisable
- The optimal pseudo-replicate count depends on total cell count per condition, so a fixed n_reps may underperform at other cell-count regimes

## Evidence

- [readme] README bootstrapping trigger: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion bootstrapping description: "we address this issue by our iteration-based bootstrapping method"
- [full_text] Pseudo-replicate split procedure: [evidence span withheld — anchor status: not_found]
- [discussion] Rare-cluster instability threshold: "we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [results] Pseudo-replicate count vs cell number: [evidence span withheld — anchor status: not_found]
