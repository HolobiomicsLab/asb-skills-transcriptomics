---
name: scrnaseq-homogeneity-detection
description: Use when a scRNA-seq dataset is about to undergo trajectory/pseudotime
  inference and you must first verify it actually contains a continuous trajectory
  (i.e., is not homogeneous cells or disconnected clusters).
when_to_use_negative:
- The dataset is already known to have a clear trajectory and you only want to rank
  embeddings — use Escort's Step 2 benchmark scoring (score > 0) instead
- The analysis goal is clustering or differential expression, not deciding whether
  trajectory inference is appropriate
- The data are non-scRNA-seq or the trajectory methods of interest do not rely on
  two-dimensional representations (Escort only considers 2D embeddings)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R/Shiny framework providing the Step 1 suitability checks HD_DCClusterscheck
    and step1_testHomogeneous
  repo: https://github.com/xiaorudong/Escort
- name: Scaffold
  role: Generates simulated scRNA-seq datasets (0% vs 20% dynamic genes) with known
    trajectory ground truth to validate detection
- name: R
  role: Implementation language for loading simulated data and running Escort's homogeneity
    checks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:c7db7fd2b99952739b929b864059e9d2d0832bfd2370c1b4abfe675510967402
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrnaseq-homogeneity-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects scRNA-seq datasets or embeddings that lack a trajectory signal — homogeneous cells or disconnected clusters — using Escort's Step 1 tests (HD_DCClusterscheck and step1_testHomogeneous) so trajectory inference is only attempted on suitable data.

## When to use

Use when a scRNA-seq dataset is about to undergo trajectory/pseudotime inference and you must first verify it actually contains a continuous trajectory (i.e., is not homogeneous cells or disconnected clusters).

## When NOT to use

- The dataset is already known to have a clear trajectory and you only want to rank embeddings — use Escort's Step 2 benchmark scoring (score > 0) instead
- The analysis goal is clustering or differential expression, not deciding whether trajectory inference is appropriate
- The data are non-scRNA-seq or the trajectory methods of interest do not rely on two-dimensional representations (Escort only considers 2D embeddings)

## Inputs

- scRNA-seq expression matrix or SingleCellExperiment loaded into R (e.g., simulated via Scaffold's simulateScaffold(), or preprocessed Seurat/SCE data)
- Cell clustering and first principal component / top 100 highly variable genes computed by the Escort pipeline

## Outputs

- Step 1 suitability flags per dataset (e.g., step1_suitability_results.csv) indicating 'suitable' or 'flagged as homogeneous/disconnected and unsuitable for trajectory fitting'
- Cluster-connection decisions based on Jaccard index scores (0.3 cutoff, >10 cells per cluster)
- Permutation-test p-values (20,000 iterations, FDR-corrected) for PC1–gene Spearman correlations

## How to apply

Run Escort's Step 1 cluster-disconnectedness check (HD_DCClusterscheck): cluster cells with scLCA, build the Manhattan distance matrix, and compute within- and between-cluster Jaccard index scores; mark two clusters as connected only if more than 10 cells per cluster meet the 0.3 Jaccard cutoff. Then run the homogeneity test (step1_testHomogeneous): compute Spearman correlations between the first principal component and a random subset of the top 100 highly variable genes, assessed with a 20,000-iteration permutation test with FDR correction, flagging homogeneity when significant correlations (p < 0.05) exceed 46% of genes. Disconnected embeddings are immediately classified as not recommended for trajectory inference. Rationale: all simulated datasets with no true trajectory (0% dynamic genes) were detected and flagged by this step, and flagged datasets yielded even less accurate trajectories than non-recommended embeddings.

## Related tools

- **Escort** (R/Shiny framework providing the Step 1 suitability checks HD_DCClusterscheck and step1_testHomogeneous) — https://github.com/xiaorudong/Escort
- **Scaffold** (Generates simulated scRNA-seq datasets (0% vs 20% dynamic genes) with known trajectory ground truth to validate detection)
- **R** (Implementation language for loading simulated data and running Escort's homogeneity checks)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Every simulated dataset with 0% dynamic genes (homogeneous, no true trajectory) is detected and flagged as unsuitable for trajectory fitting
- The linear-trajectory simulation (20% dynamic genes) is NOT falsely flagged by Step 1
- Disconnected embeddings are classified as not recommended for trajectory inference without proceeding to later Escort steps
- Jaccard connectivity decisions reproduce the thresholds: >10 cells per cluster meeting the 0.3 Jaccard cutoff
- Homogeneity flags fire only when FDR-corrected significant PC1 correlations (p < 0.05) exceed 46% of the tested highly variable genes

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Quality control steps may significantly impact Escort's embedding evaluations, so upstream preprocessing choices affect the flags

## Evidence

- [methods] cluster-disconnectedness and homogeneity flags: [evidence span withheld — anchor status: not_found]
- [methods] homogeneity permutation test thresholds: [evidence span withheld — anchor status: not_found]
- [results] validated detection of no-trajectory simulations: [evidence span withheld — anchor status: not_found]
