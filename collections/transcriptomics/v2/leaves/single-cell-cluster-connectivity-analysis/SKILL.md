---
name: single-cell-cluster-connectivity-analysis
description: Use when a scRNA-seq dataset or 2D embedding must be screened before
  trajectory fitting — e.g., before Slingshot/Monocle3 — to flag disconnected clusters
  or homogeneous cells with no trajectory signal.
when_to_use_negative:
- The trajectory method under consideration does not use two-dimensional representations
  — Escort presently only considers trajectory inference approaches that use two-dimensional
  representations.
- The embeddings derive from RNA velocity or other velocity-informed dimension reduction
  — support for these is not yet implemented in Escort (future work).
- The question is trajectory accuracy after fitting (choice of feature selection,
  dimension reduction, hyperparameters) rather than pre-fit suitability — that is
  Escort's later scoring steps, not this Step 1 check.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package providing HD_DCClusterscheck (cluster disconnectedness) and step1_testHomogeneous
    (homogeneity test) to flag datasets unsuitable for trajectory inference
  repo: https://github.com/xiaorudong/Escort
- name: scLCA
  role: Clusters cells within Escort's Step 1 disconnectedness check prior to Jaccard-based
    connectivity scoring
- name: Scaffold
  role: v0.2.0 R package used to simulate scRNA-seq data (0% vs 20% dynamic genes)
    from the human pancreas reference to validate the Step 1 flags
- name: R
  role: Implementation environment for Escort and the Step 1 checks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:7a3b24c32b2d0132b5be767d5988d7a0ad815aeb54448572a726a4a4898bb05d
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
  - scLCA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-cluster-connectivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's Step 1 suitability assessment determines whether a scRNA-seq dataset (or a given embedding) is appropriate for trajectory inference by detecting disconnected cell clusters and homogeneous cells lacking trajectory signal, using Jaccard-based cluster connectivity and a PC1–HVG correlation permutation test.

## When to use

Use when a scRNA-seq dataset or 2D embedding must be screened before trajectory fitting — e.g., before Slingshot/Monocle3 — to flag disconnected clusters or homogeneous cells with no trajectory signal.

## When NOT to use

- The trajectory method under consideration does not use two-dimensional representations — Escort presently only considers trajectory inference approaches that use two-dimensional representations.
- The embeddings derive from RNA velocity or other velocity-informed dimension reduction — support for these is not yet implemented in Escort (future work).
- The question is trajectory accuracy after fitting (choice of feature selection, dimension reduction, hyperparameters) rather than pre-fit suitability — that is Escort's later scoring steps, not this Step 1 check.

## Inputs

- Processed scRNA-seq dataset loaded into R with the Escort package (e.g., SingleCellExperiment object)
- Two-dimensional cell embeddings to evaluate (e.g., PCA, t-SNE, UMAP, diffusion maps)
- Optionally, simulated scRNA-seq data from Scaffold (simulateScaffold with estimateScaffoldParameters, human pancreas reference)

## Outputs

- Step 1 suitability flags per dataset/embedding (suitable vs. flagged as inappropriate for trajectory inference)
- Compiled flag table, e.g., step1_suitability_results.csv
- Within- and between-cluster Jaccard index scores and homogeneity test results (Spearman PC1–HVG correlations with permutation p-values)

## How to apply

Run Escort's Step 1 in R: first the cluster-disconnectedness check (HD_DCClusterscheck) — cluster cells with scLCA, build the Manhattan distance matrix, compute within- and between-cluster Jaccard index scores, and count two clusters as connected only if more than 10 cells per cluster meet the 0.3 Jaccard cutoff; any embedding found disconnected is classified immediately as not recommended for trajectory inference. Next run the homogeneity test (step1_testHomogeneous): compute Spearman correlations between the first principal component and a random subset of the top 100 highly variable genes, assess significance with a 20,000-iteration permutation test with false discovery rate correction, and flag homogeneity when significant correlations (p < 0.05) exceed 46% of genes. The rationale is that disconnected clusters or homogeneous cells indicate no underlying dynamic, so trajectory fitting would produce biased, inaccurate pseudotime. The check was validated on Scaffold v0.2.0 simulations from a human pancreas scRNA-seq reference (estimateScaffoldParameters + simulateScaffold), where every 0%-dynamic-gene (homogeneous) dataset was flagged while the 20%-dynamic-gene linear-trajectory simulation was not falsely flagged.

## Related tools

- **Escort** (R package providing HD_DCClusterscheck (cluster disconnectedness) and step1_testHomogeneous (homogeneity test) to flag datasets unsuitable for trajectory inference) — https://github.com/xiaorudong/Escort
- **scLCA** (Clusters cells within Escort's Step 1 disconnectedness check prior to Jaccard-based connectivity scoring)
- **Scaffold** (v0.2.0 R package used to simulate scRNA-seq data (0% vs 20% dynamic genes) from the human pancreas reference to validate the Step 1 flags)
- **R** (Implementation environment for Escort and the Step 1 checks)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Every simulation with no true trajectory (0% dynamic genes, homogeneous cells) is detected and flagged as unsuitable for trajectory fitting.
- The linear-trajectory simulation (20% dynamic genes) is NOT falsely flagged by Step 1.
- Any embedding found disconnected is classified immediately as not recommended for trajectory inference.
- Cluster connectivity is decided correctly: two clusters count as connected only if >10 cells per cluster meet the 0.3 Jaccard cutoff.
- Homogeneity is flagged only when significant PC1–HVG Spearman correlations (p < 0.05 after FDR over 20,000 permutations) exceed 46% of the tested top-100 HVGs.

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation.
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (an area of future work).
- Quality control steps may significantly impact Escort's embedding evaluations, so upstream QC choices affect the Step 1 flags.
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated.

## Evidence
