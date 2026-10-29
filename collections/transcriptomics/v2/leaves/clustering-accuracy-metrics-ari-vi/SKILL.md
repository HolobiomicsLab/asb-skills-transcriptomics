---
name: clustering-accuracy-metrics-ari-vi
description: Use when comparing cell-labeling methods that map target scRNA-seq/snRNA-seq
  cells to reference clusters and you need per-method coverage plus ARI/VI accuracy
  rankings against known cluster identities.
when_to_use_negative:
- When neither dataset is well clustered or contains good gene signatures — the underlying
  scID mapping is not a recommended approach, so accuracy benchmarking against it
  is not meaningful
- When the target data are log-transformed rather than library-depth normalized (TPM/CPM),
  since the mapping inputs being scored are invalid under log transformation
- When there is no reference-derived ground truth to compare predicted labels against
  — ARI/VI cannot be computed
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: scID
  role: Primary mapping method being benchmarked; assigns target cells to reference
    clusters via Fisher's Linear Discriminant Analysis-like framework, output evaluated
    with ARI/VI
  repo: https://github.com/BatadaLab/scID
- name: scmap
  role: Competing method in the benchmark; extracts features from each reference cluster
    and typically leaves a fraction of cells unassigned (lower coverage)
- name: CaSTLe
  role: Competing method in the benchmark; selects genes with high expression in both
    datasets or high mutual information with cluster identities
- name: R
  role: Implementation environment for running the three mappers and computing coverage/ARI/VI
    metrics
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:82aaa6c362d66ec8f9c169accedb8eacfa2adacec8a0e87d4faf427a108856a5
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# clustering-accuracy-metrics-ari-vi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates reference-to-target cell mapping performance (scID vs scmap vs CaSTLe) by computing assignment coverage and two independent clustering-accuracy metrics — Adjusted Rand Index (ARI) and Variation of Information (VI) — between predicted target labels and reference-derived ground truth.

## When to use

Use when comparing cell-labeling methods that map target scRNA-seq/snRNA-seq cells to reference clusters and you need per-method coverage plus ARI/VI accuracy rankings against known cluster identities.

## When NOT to use

- When neither dataset is well clustered or contains good gene signatures — the underlying scID mapping is not a recommended approach, so accuracy benchmarking against it is not meaningful
- When the target data are log-transformed rather than library-depth normalized (TPM/CPM), since the mapping inputs being scored are invalid under log transformation
- When there is no reference-derived ground truth to compare predicted labels against — ARI/VI cannot be computed

## Inputs

- Per-cell cluster label vectors for each mapping method (scID, scmap, CaSTLe) on the same target cells (e.g., scid_multiclass output scID_output$labels)
- Reference-derived ground-truth cluster identities for the target cells (reference_clusters)
- Total target cell count (e.g., 949 snRNA-seq nuclei or 288 Smart-seq2 bipolar cells)

## Outputs

- Per-method assignment coverage summary (assigned cells / total, number of clusters populated, e.g., 99.5% into 14 clusters vs 63.2% into 10)
- ARI and VI values per method comparing predicted labels to ground truth
- Summary CSV table (scid_benchmark_results.csv or coverage_metrics.csv) plus per-cell assignment lists

## How to apply

After each mapping method (scID::scid_multiclass, scmap, CaSTLe) assigns target cells to reference clusters, first tabulate assignment coverage as assigned cells / total target cells (e.g., scID 99.5% n=949 into 14 clusters for mouse brain snRNA-seq; scmap 60.2% n=574; CaSTLe 100% n=954; and in the retinal bipolar Drop-seq to Smart-seq2 comparison, scID and CaSTLe 100% 288/288 into 15 clusters vs scmap 63.2% 182/288 into 10). Then compute Adjusted Rand Index and Variation of Information between each method's predicted target labels and the reference-derived ground-truth identities, and write per-cell assignments plus a per-method coverage/ARI/VI summary table (e.g., scid_benchmark_results.csv or coverage_metrics.csv). Correct application is judged by the relative ordering: scID must attain the highest ARI / lowest VI (or be among the top two) while also reproducing its reported coverage figures.

## Related tools

- **scID** (Primary mapping method being benchmarked; assigns target cells to reference clusters via Fisher's Linear Discriminant Analysis-like framework, output evaluated with ARI/VI) — https://github.com/BatadaLab/scID
- **scmap** (Competing method in the benchmark; extracts features from each reference cluster and typically leaves a fraction of cells unassigned (lower coverage))
- **CaSTLe** (Competing method in the benchmark; selects genes with high expression in both datasets or high mutual information with cluster identities)
- **R** (Implementation environment for running the three mappers and computing coverage/ARI/VI metrics)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, normalize_reference = FALSE, estimate_weights_from_target = FALSE)
```

## Evaluation signals

- Coverage invariants must reproduce published values: scID 99.5% (n=949 into 14 clusters) and scmap 60.2% (n=574) with CaSTLe 100% (n=954) for the mouse brain snRNA-seq mapping; scID and CaSTLe 100% (288/288, 15 clusters) and scmap 63.2% (182/288, 10 clusters) for the retinal bipolar Drop-seq/Smart-seq2 mapping
- Relative accuracy ordering must be reproduced: scID attains the highest ARI / lowest VI among the three methods (or is one of the two most accurate, as in Figures 2D-2E)
- Every target cell must have a predicted label or be explicitly counted as unassigned, so coverage denominators match the total (954 or 288 cells)
- ARI and VI must both be computed as two independent accuracy metrics, not a single score
- Output summary table must contain per-method coverage, cluster count, ARI, and VI columns alongside per-cell assignments

## Limitations

- If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types, undermining the benchmark
- scmap may leave a substantial fraction of cells unassigned, so coverage differences (e.g., 60.2%-63.2%) complicate accuracy comparison on non-identical assigned subsets
- When reference and target have strong batch effect, estimation of weights from the target appears more accurate, so the estimate_weights_from_target setting changes the scID result being scored

## Evidence

- [other] Coverage and ARI/VI benchmarks (mouse brain): [evidence span withheld — anchor status: not_found]
- [other] Coverage figures (retinal bipolar): [evidence span withheld — anchor status: not_found]
