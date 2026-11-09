---
name: cluster-specific-gene-signature-extraction
description: Use when a library-depth-normalized reference scRNA-seq matrix (genes
  x cells) with cluster labels is available and you need per-cluster marker gene signatures
  (e.g. logFC >= 0.6) to feed scid_multiclass for cross-dataset cell type mapping.
when_to_use_negative:
- The reference data are not well clustered and lack good gene signatures — the article
  states that in this case scID is not a recommended approach for identification of
  equivalent cell types
- The input matrix is already log-transformed — scID requires library-depth (TPM/CPM)
  normalization only
- You only need single known biomarkers to identify cell types — the paper notes biomarker-based
  identification works poorly owing to the dropout problem in scRNA-seq data, so full
  cluster-specific signatures (not individual markers) are required
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scID
  role: Provides find_markers, counts_to_cpm, gmt_to_markers, and scID_weight functions
    for extracting cluster-specific signatures and estimating Fisher's-LDA-based discriminative
    weights
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Implementation environment in which scID's find_markers and normalization
    functions are run
- name: Seurat
  role: Imported by scID; typical source of reference_clusters from prior clustering
    of the reference data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:e77f8f6be483bbcb3889a9eda74c21ce8f68b094fee43451f74f82edf9b907ce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  repo_url: https://github.com/BatadaLab/scID
  tools:
  - R
  - Seurat
  - mclust
  - scID
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cluster-specific-gene-signature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract cluster-specific gene signatures (differentially expressed gene sets) from a reference scRNA-seq dataset with known cluster labels, as the first stage of the scID pipeline for identifying transcriptionally equivalent cells across datasets with batch effect.

## When to use

Use when a library-depth-normalized reference scRNA-seq matrix (genes x cells) with cluster labels is available and you need per-cluster marker gene signatures (e.g. logFC >= 0.6) to feed scid_multiclass for cross-dataset cell type mapping.

## When NOT to use

- The reference data are not well clustered and lack good gene signatures — the article states that in this case scID is not a recommended approach for identification of equivalent cell types
- The input matrix is already log-transformed — scID requires library-depth (TPM/CPM) normalization only
- You only need single known biomarkers to identify cell types — the paper notes biomarker-based identification works poorly owing to the dropout problem in scRNA-seq data, so full cluster-specific signatures (not individual markers) are required

## Inputs

- Raw or library-depth-normalized reference gene expression matrix (counts_to_cpm can convert raw counts; log-transformed data are not allowed)
- Named list/vector of cluster labels for the reference cells (reference_clusters), e.g. from prior Seurat clustering
- Optional GMT file of pre-defined signatures (convertible via gmt_to_markers)
- Parameters: logFC minimum threshold (e.g. 0.6), only.pos flag, normalize_reference flag

## Outputs

- markers data frame of cluster-specific signature genes (scID_output$markers), containing positive and, if only_pos = FALSE, negative marker genes per reference cluster
- Downstream: per-cluster gene weights (scID_output$weights) estimated by scID_weight from positive/negative training cells
- Optional sanity-check heatmap of average signature-gene expression per cluster (make_heatmap)

## How to apply

First normalize the raw count matrix to library-depth-normalized expression (TPM or CPM) using scID's counts_to_cpm/normalize_gene functions; log-transformed data are explicitly not compatible with scID. Then run find_markers(reference_gem, reference_clusters, logFC, only.pos, normalize_reference), where logFC sets the minimum log-fold-change for a gene to be selected as cluster-specific (the vignette uses logFC = 0.6). Set only.pos = FALSE when you want to retain downregulated cluster-specific genes as well — scID v2.0.0 uses negative markers together with positive markers, which improves classification in the presence of very similar cell types. The resulting markers data frame is passed directly to scid_multiclass (or converted from a GMT file with gmt_to_markers), where it also drives weight estimation via choose_training_set and scID_weight. If reference and target have strong batch effect, weights should be estimated from the target data (estimate_weights_from_target = TRUE), but the signature extraction itself is always performed on the reference clusters.

## Related tools

- **scID** (Provides find_markers, counts_to_cpm, gmt_to_markers, and scID_weight functions for extracting cluster-specific signatures and estimating Fisher's-LDA-based discriminative weights) — https://github.com/BatadaLab/scID
- **R** (Implementation environment in which scID's find_markers and normalization functions are run)
- **Seurat** (Imported by scID; typical source of reference_clusters from prior clustering of the reference data)

## Examples

```
markers <- scID::find_markers(reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, only.pos = FALSE, normalize_reference = TRUE)
```

## Evaluation signals

- Every returned signature gene passes the minimum log-fold-change cutoff (e.g. logFC = 0.6) relative to other reference clusters
- When only.pos = FALSE, the markers set includes both upregulated (positive) and downregulated (negative) genes per cluster, not just positive markers
- Input gene expression was normalized to CPM/TPM and is not log-transformed before find_markers
- A make_heatmap of average signature-gene expression shows expected cluster-specific (enriched) expression patterns for the extracted genesets
- Signatures extracted from the same dataset used as both reference and target yield relatively high self-mapping Adjusted Rand Index values (mclust::adjustedRandIndex, close to 1, as in Figure 1D)

## Limitations

- Signature quality depends on the reference clustering being reliable; if neither dataset is well clustered and contains good gene signatures, scID is not recommended
- Gene signature identification is affected by the dropout problem inherent to scRNA-seq data
- For highly shallow target data, gene expression imputation is only proposed as future work and is not implemented in scID
- No specific software version for the signature-extraction functions is stated beyond the v2.0.0 release (May 17, 2019) that added negative markers

## Evidence

- [readme] v2.0.0 uses negative markers: [evidence span withheld — anchor status: external_doc]
