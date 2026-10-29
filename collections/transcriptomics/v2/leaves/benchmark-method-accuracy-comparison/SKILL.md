---
name: benchmark-method-accuracy-comparison
description: Use when comparing label-transfer methods (e.g., scID vs scmap, CaSTLe,
  scVI) that map cells from a target scRNA-seq dataset to clusters of a reference
  dataset, and you need coverage and accuracy metrics on a common benchmark.
when_to_use_negative:
- The goal is unsupervised batch integration or correction of datasets rather than
  benchmarking supervised cell-type label transfer against known labels
- Neither the reference nor the target data are well clustered or contain good gene
  signatures, so no mapping method produces a meaningful assignment to benchmark
- Input matrices are already log-transformed, since scID requires library-depth (TPM/CPM)
  normalized but non-log-transformed data
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scID
  role: Primary mapping method under benchmark; runs find_markers and scid_multiclass
    with estimate_weights_from_target to assign target cells to reference clusters
  repo: https://github.com/BatadaLab/scID
- name: scmap
  role: Competing mapping method; extracts features from each reference cluster and
    leaves a fraction of target cells unassigned, providing a coverage baseline
- name: CaSTLe
  role: Competing method; selects genes with high expression in both datasets or high
    mutual information with cluster identities to assign target cells
- name: scVI
  role: Integration-based competing method applied to the same reference-target pairs
- name: R
  role: Implementation environment for loading matrices, running all methods, and
    computing ARI/VI metrics
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:a1c52f0ae1ca407497de5981c9ec20f1672f74b109ec5477d0a54afddf52b275
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
  - scID
  - scVI
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-method-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks supervised cell-type mapping methods (scID, scmap, CaSTLe, scVI) on the same reference-target scRNA-seq pairs by computing per-method assignment coverage and clustering-accuracy metrics (ARI, VI) against published cell-type labels.

## When to use

Use when comparing label-transfer methods (e.g., scID vs scmap, CaSTLe, scVI) that map cells from a target scRNA-seq dataset to clusters of a reference dataset, and you need coverage and accuracy metrics on a common benchmark.

## When NOT to use

- The goal is unsupervised batch integration or correction of datasets rather than benchmarking supervised cell-type label transfer against known labels
- Neither the reference nor the target data are well clustered or contain good gene signatures, so no mapping method produces a meaningful assignment to benchmark
- Input matrices are already log-transformed, since scID requires library-depth (TPM/CPM) normalized but non-log-transformed data

## Inputs

- Reference scRNA-seq gene expression matrix (NxM, e.g. Segerstolpe Smart-Seq2 pancreas or ~26,800-cell Drop-seq retinal bipolar) with published cluster labels
- Target scRNA-seq gene expression matrix (nxm, e.g. Baron CEL-Seq1, Muraro CEL-Seq2, or 288-cell Smart-seq2 bipolar cells) with published cell-type labels as ground truth
- Library-depth normalized (TPM/CPM, non-log-transformed) count matrices in R data frames with unique cell names
- List of reference cluster labels (reference_clusters)

## Outputs

- Per-method per-target mapping accuracy table (e.g. coverage_metrics.csv) with coverage (assigned cells / total), Adjusted Rand Index, and Variation of Information
- Per-cell cluster assignments from each method (scID_output$labels, scmap, CaSTLe assignments)
- Comparison table/figure mirroring Figure 1E or Figures 2D-2E showing relative method ranking
- scID outputs: list of labels, markers data frame, weights, and target-cell scores

## How to apply

For each reference-target pair, run all competing methods on the same inputs: library-depth normalize (TPM or CPM) both count matrices without log-transformation, since scID is incompatible with log-transformed data. Extract cluster-specific gene signatures from the reference with scID::find_markers (logFC threshold), then run scid_multiclass on the target with estimate_weights_from_target = TRUE, which is more accurate under strong batch effect (e.g., Smart-Seq2 reference vs CEL-Seq1/CEL-Seq2 targets, or Drop-seq reference vs Smart-seq2 target). Run scmap (extracts features from each reference cluster) and CaSTLe (selects genes with high expression in both datasets or high mutual information with cluster identities) on the same pairs; run scVI as the integration-based competitor. For each method compute assignment coverage (assigned cells / total target cells, e.g. 182/288 = 63.2% for scmap on the retinal bipolar Smart-seq2 target) and clustering-accuracy metrics — Adjusted Rand Index and Variation of Information — against published cell-type labels. Assemble per-method, per-target accuracy into a summary table and figure (mirroring Figures 1E and 2D-2E) where a method is judged by ranking within the top two in both accuracy and consistency across targets.

## Related tools

- **scID** (Primary mapping method under benchmark; runs find_markers and scid_multiclass with estimate_weights_from_target to assign target cells to reference clusters) — https://github.com/BatadaLab/scID
- **scmap** (Competing mapping method; extracts features from each reference cluster and leaves a fraction of target cells unassigned, providing a coverage baseline)
- **CaSTLe** (Competing method; selects genes with high expression in both datasets or high mutual information with cluster identities to assign target cells)
- **scVI** (Integration-based competing method applied to the same reference-target pairs)
- **R** (Implementation environment for loading matrices, running all methods, and computing ARI/VI metrics)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem, reference_gem, reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- On the retinal bipolar benchmark, scID and CaSTLe must assign 100% of the 288 Smart-seq2 cells into 15 Drop-seq clusters while scmap covers only 63.2% (182/288 cells into 10 clusters)
- ARI and VI values must reproduce the relative accuracy ordering of Figures 2D-2E
- scID must rank within the top two methods in both accuracy and consistency across the Baron CEL-Seq1 and Muraro CEL-Seq2 targets, matching the qualitative outcome of Figure 1E
- The scid_multiclass output must be a list of four objects: labels, markers, weights, and scores
- A per-method coverage/ARI/VI summary table (e.g. coverage_metrics.csv) is produced for every reference-target pair

## Limitations

- If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types
- Imputation of gene expression, particularly when the target scRNA-seq is highly shallow, is deferred to future work and not implemented or evaluated
- When reference and target are strongly imbalanced in number of cells and library size per cell, how batch correction methods perform is unclear, and untangling technical from biological variation risks over-correction and loss of biological variance
- Log-transformed data are not compatible with scID; only library-depth normalization (e.g., TPM, CPM) is supported

## Evidence
