---
name: batch-effect-robust-cell-annotation
description: Use when a target scRNA-seq dataset (e.g. Smart-seq2) must be annotated
  against a clustered reference (e.g. Drop-seq) across technologies with batch effect
  and strongly imbalanced cell numbers or library sizes per cell.
when_to_use_negative:
- When neither dataset is well clustered or contains good gene signatures — the article
  states scID is not a recommended approach for identification of equivalent cell
  types in that case
- When the target data is highly shallow and imputation of gene expression is required
  — this is deferred to future work and not implemented or evaluated
- When log-transformed data is the only available input — scID requires non-log-transformed,
  library-depth normalized expression
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scID
  role: 'Primary method: extracts cluster-specific gene signatures via find_markers
    and assigns target cells to reference clusters via scid_multiclass with Fisher''s
    Linear Discriminant Analysis-like weighting'
  repo: https://github.com/BatadaLab/scID
- name: scmap
  role: Competing mapping baseline that extracts features from each reference cluster
    and leaves a fraction of cells unassigned (63.2% coverage on the retinal bipolar
    task)
- name: CaSTLe
  role: Competing baseline that selects genes with high expression in both target
    and reference data or high mutual information with cluster identities (100% coverage,
    15 clusters)
- name: R
  role: Implementation environment; scID is installed from GitHub with devtools and
    run via scid_multiclass in R
  repo: https://github.com/BatadaLab/scID
- name: Seurat
  role: Listed as an Imports dependency of the scID package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:bf7905ec689b4c37e0c871d5e651ebcdf96858a4a3d8b829418cfef35321596d
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
  - CaSTLe
  - R
  - Seurat
  - scID
  - scmap
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# batch-effect-robust-cell-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map cells from a target scRNA-seq dataset to reference clusters using scID's Fisher's Linear Discriminant Analysis-like framework, which exploits cluster-specific gene signatures and is robust to batch effect and imbalance in cell number and library size, as demonstrated on the Drop-seq retinal bipolar reference (~26,800 cells, 15 clusters) mapping to a 288-cell Smart-seq2 target.

## When to use

Use when a target scRNA-seq dataset (e.g. Smart-seq2) must be annotated against a clustered reference (e.g. Drop-seq) across technologies with batch effect and strongly imbalanced cell numbers or library sizes per cell.

## When NOT to use

- When neither dataset is well clustered or contains good gene signatures — the article states scID is not a recommended approach for identification of equivalent cell types in that case
- When the target data is highly shallow and imputation of gene expression is required — this is deferred to future work and not implemented or evaluated
- When log-transformed data is the only available input — scID requires non-log-transformed, library-depth normalized expression

## Inputs

- Reference gene expression matrix (NxM data frame, N genes x M reference cells), library-depth or column normalized (TPM/CPM), non-log-transformed
- Target gene expression matrix (nxm data frame, n genes x m target cells), library-depth or column normalized, unique cell names
- Named list of cluster labels for the reference cells (e.g. 15 Drop-seq clusters)
- logFC threshold for cluster-specific gene selection

## Outputs

- scID_output$labels: named list of assigned cluster labels for every target cell
- scID_output$markers: data frame of signature genes extracted from the reference clusters
- scID_output$weights: list of estimated weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores for each reference cluster-specific geneset
- Per-method coverage and ARI/VI accuracy summary (e.g. coverage_metrics.csv)
- Optional heatmap of average expression of cluster-specific genesets (make_heatmap) and 2D marker score plot (plot_score_2D)

## How to apply

Load library-depth normalized (TPM/CPM), non-log-transformed expression matrices for reference and target, with cluster labels for the reference cells. Extract cluster-specific gene signatures from the reference with scID::find_markers using a logFC threshold (only.pos), then run scID::scid_multiclass with estimate_weights_from_target=TRUE so discriminative gene weights are estimated from the target data — more accurate under strong batch effect. Target cells are scored on positive and negative signature genes and assigned via a finite mixture of Gaussians, giving every cell a cluster label. Compare against scmap (which leaves cells unassigned) and CaSTLe by computing per-method coverage (assigned cells / 288) and accuracy via Adjusted Rand Index and Variation of Information against reference-derived ground-truth identities. On the retinal bipolar task, scID and CaSTLe reached 100% coverage (288/288 cells, 15 clusters) versus scmap's 63.2% (182/288, 10 clusters), with scID among the two most accurate methods.

## Related tools

- **scID** (Primary method: extracts cluster-specific gene signatures via find_markers and assigns target cells to reference clusters via scid_multiclass with Fisher's Linear Discriminant Analysis-like weighting) — https://github.com/BatadaLab/scID
- **scmap** (Competing mapping baseline that extracts features from each reference cluster and leaves a fraction of cells unassigned (63.2% coverage on the retinal bipolar task))
- **CaSTLe** (Competing baseline that selects genes with high expression in both target and reference data or high mutual information with cluster identities (100% coverage, 15 clusters))
- **R** (Implementation environment; scID is installed from GitHub with devtools and run via scid_multiclass in R) — https://github.com/BatadaLab/scID
- **Seurat** (Listed as an Imports dependency of the scID package)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- Coverage: scID must assign 100% of target cells (288/288 into 15 Drop-seq clusters); scmap should leave cells unassigned (63.2%, 182/288 into 10 clusters)
- Accuracy ordering: Adjusted Rand Index and Variation of Information must reproduce the relative accuracy ordering of Figures 2D–2E, with scID among the two most accurate methods (with CaSTLe)
- Output schema check: scid_multiclass returns a list with exactly four objects (labels, markers, weights, scores)
- Input check: expression entries are library-depth (TPM/CPM) column-normalized and NOT log-transformed
- Sanity visualizations: make_heatmap of signature geneset expression across clusters and plot_score_2D projection of selected target cells onto positive/negative marker scores

## Limitations

- If neither of the two datasets is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types
- Imputation of gene expression, particularly when the target scRNA-seq is highly shallow, may increase power but is deferred to future work and not implemented or evaluated
- No software version is stated for scID in the discussion section
- scID requires non-log-transformed, library-depth normalized data; log-transformed inputs are incompatible

## Evidence

- [abstract] scID uses Fisher's LDA-like framework across datasets with batch effect: "we developed scID (Single Cell IDentification), which uses the Fisher's Linear Discriminant Analysis-like framework to identify transcriptionally related cell types between scRNA-seq datasets"
- [intro] scID 3-stage workflow: signatures, weights, mixture-of-Gaussians assignment: "A finite mixture of Gaussians is used to model the scores of the target cells. Cells with high likelihood of falling under the distribution with the highest average score are assigned to the"
- [discussion] Estimating weights from the target is more accurate under strong batch effect: "When the reference and the target data have strong batch effect, estimation of weights from the target appears to be more accurate."
- [methods] Input normalization requirement: library-depth normalized, not log-transformed: "Any library-depth normalization (e.g. TPM, CPM) is compatibe with scID, but not log-transformed data."
- [methods] Core scid_multiclass invocation with estimate_weights_from_target: "estimate_weights_from_target is set to TRUE in order to estimated gene weights from the target by selecting training target cells"
- [intro] scmap baseline leaves cells unassigned: "To map cells from target scRNA-seq data to a reference data, scmap (Kiselev et al., 2018) extracts features from each reference cluster"
- [readme] README: input format and scid_multiclass usage for target cells: "scID_output <- scID::scid_multiclass(target_gem, reference_gem, reference_clusters, ...)"
- [readme] README: v2.0.0 uses negative markers together with positive: [evidence span withheld — anchor status: external_doc]
