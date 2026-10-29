---
name: single-cell-pseudotime-trajectory-inference
description: Use when you have a single-cell scRNA-seq count matrix with a differentiation
  hierarchy (e.g., basal-to-differentiated epithelium) and need to root a trajectory
  objectively by pluripotency scores rather than a priori marker assumptions.
when_to_use_negative:
- The goal is only pluripotency/cell-annotation scoring (use ORIGINS/ORIGINS2 activity
  scores alone) and no trajectory or lineage structure is needed.
- The input is not a single-cell count matrix with an expected differentiation hierarchy
  (e.g., a bulk transcriptome or fully differentiated-only sample), so root selection
  by activity scores is not meaningful.
- A biologically validated root is already available and only the pseudotime computation
  is required; ORIGINS-based rooting adds an unnecessary step.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Computes per-cell differentiation PPI network activity scores used to select
    the highest-pluripotency cells within the basal cluster as the trajectory root
  repo: https://github.com/danielasenraoka/ORIGINS
- name: remotes
  role: R package required to install ORIGINS from GitHub via remotes::install_github()
- name: Slingshot
  role: Performs the pseudotime trajectory inference using the ORIGINS-selected root
    cells, yielding branching lineages
- name: R
  role: Runtime environment for the ORIGINS package, Slingshot, and tradeSeq workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:3cab9f0b5b73453e57b044bb3c7771bd943666b5ae9933b2817ae025bb3b4139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - R
  - RTN
  - Seurat
  - Slingshot
  - TimeTalk
  - monocle3
  - remotes
  - scPlant
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-trajectory-pseudotime-inference
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-pseudotime-trajectory-inference

## Summary

Run pseudotime trajectory inference on scRNA-seq data using ORIGINS-derived differentiation PPI network activity to select stem/progenitor cells as the trajectory root, then infer lineages (e.g., Slingshot) that branch from the root cluster into terminal cell types.

## When to use

Use when you have a single-cell scRNA-seq count matrix with a differentiation hierarchy (e.g., basal-to-differentiated epithelium) and need to root a trajectory objectively by pluripotency scores rather than a priori marker assumptions.

## When NOT to use

- The goal is only pluripotency/cell-annotation scoring (use ORIGINS/ORIGINS2 activity scores alone) and no trajectory or lineage structure is needed.
- The input is not a single-cell count matrix with an expected differentiation hierarchy (e.g., a bulk transcriptome or fully differentiated-only sample), so root selection by activity scores is not meaningful.
- A biologically validated root is already available and only the pseudotime computation is required; ORIGINS-based rooting adds an unnecessary step.

## Inputs

- normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g., breast epithelium counts from GEO accession GSE113197
- cluster labels for cells (e.g., basal cluster assignment used to restrict root selection)
- differentiation PPIN edges (differentiation_edges, bundled with the ORIGINS R package)

## Outputs

- per-cell differentiation network activity scores (diff_activity vector)
- inferred pseudotime trajectory with lineages rooted at the highest-activity cells, e.g., Lineage 1 (basal to L1) and Lineage 2 (basal to L2)
- optional lineage-specific differential expression results from tradeSeq
- trajectory figure (e.g., slingshot_trajectory_breast.png)

## How to apply

Load the normalized count matrix (rows = genes, columns = cells; e.g., breast epithelium dataset GSE113197) and compute per-cell differentiation network activity with ORIGINS via `diff_activity <- activity(expression_matrix, differentiation_edges)`, where the differentiation PPIN is bundled with the package. Within the putative origin cluster (e.g., the basal cluster), select the cells with the highest differentiation PPI activity scores and designate them as the trajectory root, since ORIGINS was originally created to identify stem and progenitor cells as origins for trajectory inference. Run Slingshot pseudotime inference with this root and inspect the inferred lineage structure. Verify the topology matches the expected biology: the trajectory progresses from the basal root through a shared intermediate state and bifurcates into L1 (Lineage 1) and L2 (Lineage 2). Optionally apply tradeSeq to the two lineages to test lineage-specific differential expression along pseudotime.

## Related tools

- **ORIGINS** (Computes per-cell differentiation PPI network activity scores used to select the highest-pluripotency cells within the basal cluster as the trajectory root) — https://github.com/danielasenraoka/ORIGINS
- **remotes** (R package required to install ORIGINS from GitHub via remotes::install_github())
- **Slingshot** (Performs the pseudotime trajectory inference using the ORIGINS-selected root cells, yielding branching lineages)
- **R** (Runtime environment for the ORIGINS package, Slingshot, and tradeSeq workflow)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- The activity() call runs without error on a genes-by-cells matrix and returns a per-cell activity vector (no normalization parameter in ORIGINS; ORIGINS2 accepts a logical normalization flag).
- The selected root cells are exactly the highest differentiation-activity cells within the basal cluster, not in L1/L2 clusters.
- The inferred Slingshot trajectory bifurcates from the basal root into exactly two lineages (Lineage 1: basal to L1; Lineage 2: basal to L2) through a shared intermediate state.
- Dataset accession matches the article benchmark (GSE113197 for breast epithelial cells) if reproducing the published result.
- Optionally, tradeSeq reports lineage-specific differential expression along pseudotime on both inferred lineages.

## Limitations

- The trajectory result depends on the quality of the cluster labels; the root is chosen only among the highest-activity cells within a predefined cluster (basal), so misclustering propagates into rooting.
- ORIGINS activity is a network-based score over the differentiation PPIN; non-human or poorly covered tissues may have incomplete PPIN coverage, and other biological processes require the extended ORIGINS2 package or PyOrigins.
- The article benchmarks only four healthy human datasets (breast, colon, hematopoietic bone marrow, lung); behavior on other tissues or disease states is not validated here.

## Evidence

- [readme] This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference.: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [abstract] We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set.: "We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set."
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
