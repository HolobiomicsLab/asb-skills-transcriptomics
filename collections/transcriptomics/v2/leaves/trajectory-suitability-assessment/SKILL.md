---
name: trajectory-suitability-assessment
description: Use when starting a scRNA-seq trajectory analysis and you need to decide
  whether fitting a trajectory is appropriate — e.g. datasets may be homogeneous or
  contain disconnected cell populations with no underlying dynamic.
when_to_use_negative:
- The dataset already has a known continuous trajectory with ground truth pseudotime
  — proceed directly to embedding evaluation rather than re-testing suitability
- The trajectory inference method does not use two-dimensional representations; Escort
  only considers approaches based on two-dimensional embeddings
- Input is raw unprocessed counts — quality control, filtering of genes with fewer
  than three total counts, and LogNormalize normalization (Seurat NormalizeData())
  should precede the suitability assessment
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Escort
  role: R package / Shiny app implementing the Step 1 disconnected-cluster (HD_DCClusterscheck)
    and homogeneity (step1_testHomogeneous) suitability tests
  repo: https://github.com/xiaorudong/Escort
- name: Scaffold
  role: R package v0.2.0 used to simulate trajectory-free (0% dynamic genes) and linear-trajectory
    (20% dynamic genes) scRNA-seq datasets from a human pancreas reference to validate
    Step 1 flagging
- name: R
  role: Implementation environment for Escort's suitability checks and simulation
    workflow
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:5afaefa96b7cd325180e4745d1e1a99d319eddd1db0de2e56f61e192d5862541
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# trajectory-suitability-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's Step 1 determines whether a scRNA-seq dataset is suitable for trajectory inference at all, by testing for disconnected cell clusters and homogeneous cells lacking trajectory signal before any embedding or pseudotime estimation is attempted.

## When to use

Use when starting a scRNA-seq trajectory analysis and you need to decide whether fitting a trajectory is appropriate — e.g. datasets may be homogeneous or contain disconnected cell populations with no underlying dynamic.

## When NOT to use

- The dataset already has a known continuous trajectory with ground truth pseudotime — proceed directly to embedding evaluation rather than re-testing suitability
- The trajectory inference method does not use two-dimensional representations; Escort only considers approaches based on two-dimensional embeddings
- Input is raw unprocessed counts — quality control, filtering of genes with fewer than three total counts, and LogNormalize normalization (Seurat NormalizeData()) should precede the suitability assessment

## Inputs

- Normalized scRNA-seq expression matrix (e.g. SingleCellExperiment, such as Escort's exampleData_linear dataset)
- Cell count matrix or cluster assignments for Jaccard-based connectivity scoring
- Top 100 highly variable genes identified e.g. with modelGeneVar()

## Outputs

- Step 1 suitability flags indicating whether the dataset is appropriate for trajectory inference (e.g. step1_suitability_results.csv)
- Per-cluster Jaccard index scores and connected/disconnected cluster classification
- Homogeneity test result: fraction of significantly PC1-correlated HVGs vs the 46% threshold

## How to apply

First run the cluster-disconnectedness check (HD_DCClusterscheck): cluster cells with scLCA (or SC3 when prior cell type information exists), compute the N x N Manhattan distance matrix between cells, derive within- and between-cluster Jaccard index scores from those distance distributions, and mark two clusters as connected only if more than 10 cells per cluster meet the 0.3 Jaccard cutoff. Then run the homogeneity test (step1_testHomogeneous): compute Spearman correlations between the first principal component and a random subset of the top 100 highly variable genes, assess significance with a 20,000-iteration permutation test with false discovery rate correction, and flag homogeneity when significant correlations (p < 0.05) exceed 46% of the genes. Compile the resulting suitability flags; datasets flagged as disconnected or homogeneous should not proceed to embedding evaluation (Escort Steps 2-3). The rationale is that simulated datasets with no true trajectory (0% dynamic genes) are reliably detected and flagged, preventing biased trajectory fitting on trajectory-free data.

## Related tools

- **Escort** (R package / Shiny app implementing the Step 1 disconnected-cluster (HD_DCClusterscheck) and homogeneity (step1_testHomogeneous) suitability tests) — https://github.com/xiaorudong/Escort
- **Scaffold** (R package v0.2.0 used to simulate trajectory-free (0% dynamic genes) and linear-trajectory (20% dynamic genes) scRNA-seq datasets from a human pancreas reference to validate Step 1 flagging)
- **R** (Implementation environment for Escort's suitability checks and simulation workflow)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Every simulated dataset with no true trajectory (0% dynamic genes from Scaffold) must be detected and flagged as unsuitable by Step 1
- A simulated linear-trajectory dataset (20% dynamic genes) must NOT be falsely flagged as unsuitable
- Two clusters are declared connected only when more than 10 cells per cluster meet the 0.3 Jaccard cutoff — verify this threshold governs the classification
- Homogeneity is flagged only when significant Spearman correlations (p < 0.05, 20,000-iteration permutation test, FDR corrected) exceed 46% of the top 100 HVGs
- Output table (e.g. step1_suitability_results.csv) compiles flags for all tested datasets with no missing entries

## Limitations

- Flagged (non-recommended) datasets are not guaranteed to be excluded from further analysis by downstream tools — flagged datasets in the simulation study still produced trajectories, only with worse accuracy
- Escort's Step 1 timing scales with dataset size: ~5 min for 1000 cells and ~18 min for 3000 cells on laptop-class hardware (16 GB RAM), so very large datasets may be slow
- Escort only considers trajectory inference approaches that use two-dimensional representations, so suitability flags are tied to that restriction
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated

## Evidence
