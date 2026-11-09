---
name: cross-dataset-batch-effect-evaluation
description: Use when comparing reference-based cell-type mapping methods across two
  single-cell datasets with batch effect and strong asymmetry in cell number and library
  depth (e.g., whole-cell scRNA-seq reference to ultra-sparse snRNA-seq nuclei target).
when_to_use_negative:
- When neither dataset is well clustered and contains good gene signatures — the article
  states 'If neither of the two data is well clustered and contains good gene signatures,
  scID is not a recommended approach for identification of equivalent cell types.'
- When the goal is integration via batch correction of imbalanced datasets — the article
  notes it is unclear how batch correction methods perform when data are significantly
  imbalanced in cell number and library size per cell
- When a biomarker-list-only approach would suffice — biomarker-based identification
  does not work effectively owing to the dropout problem with scRNA-seq data, so a
  drop-out-tolerant scorer is needed instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: Primary method under evaluation; assigns target cells to reference clusters
    via Fisher's-LDA-like weighted signatures and a finite mixture of Gaussians
  repo: https://github.com/BatadaLab/scID
- name: scmap
  role: Baseline comparator that extracts features from each reference cluster to
    map target cells
- name: CaSTLe
  role: Baseline comparator using genes with high expression in both datasets or high
    mutual information with cluster identity
- name: Seurat
  role: R ecosystem dependency listed in scID's Imports for handling single-cell objects
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:acda3e365db5233ecbc871547a0ade5748c14ae552984313aed59cef6f8cdfa7
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-dataset-batch-effect-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scID's Fisher's-LDA-based cell mapping against scmap and CaSTLe on a batch-affected scRNA-seq reference-to-target task (E18 mouse brain scRNA-seq vs. snRNA-seq nuclei), measuring assignment coverage plus ARI/VI accuracy against reference-derived ground truth.

## When to use

Use when comparing reference-based cell-type mapping methods across two single-cell datasets with batch effect and strong asymmetry in cell number and library depth (e.g., whole-cell scRNA-seq reference to ultra-sparse snRNA-seq nuclei target).

## When NOT to use

- When neither dataset is well clustered and contains good gene signatures — the article states 'If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types.'
- When the goal is integration via batch correction of imbalanced datasets — the article notes it is unclear how batch correction methods perform when data are significantly imbalanced in cell number and library size per cell
- When a biomarker-list-only approach would suffice — biomarker-based identification does not work effectively owing to the dropout problem with scRNA-seq data, so a drop-out-tolerant scorer is needed instead

## Inputs

- target_gem.rds — TPM-normalized target gene expression matrix (genes x cells, not log-transformed)
- reference_gem.rds — TPM-normalized reference gene expression matrix with known clusters
- reference_clusters.rds — named list of reference cluster labels serving as ground truth
- Two 10X E18 mouse brain datasets: neuron_9k (single cells) as reference and nuclei_900 (single nuclei) as target

## Outputs

- scid_benchmark_results.csv — per-method coverage (fraction of target cells assigned and number of clusters hit) and ARI/VI values
- Per-method predicted cluster labels for all target nuclei (e.g., scID_output$labels written to scid_labels.csv)

## How to apply

Load pre-computed TPM-normalized matrices (target_gem.rds, reference_gem.rds, reference_clusters.rds) from the BatadaLab/scID ExampleData folder, confirming they are library-depth normalized (TPM/CPM) and not log-transformed, as required by scID. Run scID::scid_multiclass with logFC = 0.6, only_pos = FALSE (include downregulated cluster-specific genes), normalize_reference = FALSE, and estimate_weights_from_target = FALSE since the datasets' library depths are similar. Map the same target cells with scmap (extracts features from each reference cluster) and with CaSTLe (using genes with high expression in both datasets or high mutual information with cluster identity). Compute Adjusted Rand Index and Variation of Information between predicted labels and the reference-derived ground truth for all three methods, then tabulate coverage (scID 99.5%, n=949 into 14 clusters; scmap 60.2%, n=574 into 11; CaSTLe 100%, n=954 into 15) and write results to scid_benchmark_results.csv.

## Related tools

- **scID** (Primary method under evaluation; assigns target cells to reference clusters via Fisher's-LDA-like weighted signatures and a finite mixture of Gaussians) — https://github.com/BatadaLab/scID
- **scmap** (Baseline comparator that extracts features from each reference cluster to map target cells)
- **CaSTLe** (Baseline comparator using genes with high expression in both datasets or high mutual information with cluster identity)
- **Seurat** (R ecosystem dependency listed in scID's Imports for handling single-cell objects)

## Examples

```
scID_output <- scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, only_pos = FALSE, normalize_reference = FALSE, estimate_weights_from_target = FALSE)
```

## Evaluation signals

- scID must attain the highest ARI and lowest VI among scID, scmap, and CaSTLe
- scID's assignment coverage must reproduce the reported 99.5% (949 of 954 target nuclei assigned into 14 reference clusters)
- make_heatmap of cluster-specific geneset expression: the target heatmap pattern should resemble the reference heatmap
- No target cells should be assigned to reference clusters 5 and 13 (gray columns in the heatmap)
- Coverage values must match: scID 99.5% (n=949, 14 clusters), scmap 60.2% (n=574, 11 clusters), CaSTLe 100% (n=954, 15 clusters)

## Limitations

- When the reference and target data have strong batch effect, estimating gene signature weights from the target (estimate_weights_from_target = TRUE) appears more accurate than from the reference
- Imputation of gene expression for shallow target data was deferred to future work, not implemented or evaluated
- Untangling technical from biological variation between datasets may not be possible and risks over-correction, losing biological variance

## Evidence

- [results] Coverage benchmark headline: [evidence span withheld — anchor status: not_found]
