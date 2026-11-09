---
name: cell-type-silhouette-and-batch-entropy-metrics
description: Use when comparing cell-type annotation or integration methods on the
  same labeled multi-batch scRNA-seq datasets and you need standardized scores (silhouette,
  batch entropy, F1, accuracy) plus statistical significance.
when_to_use_negative:
- Data have no reference cell-type labels — cell-type silhouette, macro F1, and accuracy
  cannot be computed without ground-truth annotations
- Data have no batch or condition annotation — batch entropy mixing cannot be defined
- You need de novo discovery of unseen cell types rather than comparing methods on
  shared label sets (the MASI paper notes MASI alone cannot annotate unseen cell types;
  SCCAF is needed for subtypes)
tools:
- name: MASI
  role: One benchmarked annotation/integration method whose outputs are scored; produces
    cell-type scores and labels via masi.gene2cell and masi.parallel
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Underlying single-cell Python framework in which the benchmark datasets and
    downstream metric computation operate
- name: Symphony
  role: Baseline reference-mapping method (variable gene selection, scaling, PCA,
    Harmony batch correction, query mapping) whose outputs are scored by these metrics
  repo: https://github.com/immunogenomics/symphony
- name: scNym
  role: Baseline semi-supervised adversarial neural-network label-transfer method
    scored by these metrics with its default pipeline
  repo: https://github.com/calico/scnym
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:8d24e202b6264a9dff0e14ab53ecda17c77e9852975e425eedb977ce48591f02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - Harmony
  - LIGER
  - MACA
  - MASI
  - Scanorama
  - Scanpy
  - Seurat
  - Symphony
  - scArches
  - scNym
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# Cell-type silhouette and batch entropy metrics

## Summary

Quantitative evaluation suite for scRNA-seq label-transfer/integration benchmarks: compute macro F1, overall accuracy, cell-type silhouette (separation of cell types), and batch entropy (batch mixing) per method, then aggregate into annotation, integration, and overall performance scores and test differences with paired t-tests.

## When to use

Use when comparing cell-type annotation or integration methods on the same labeled multi-batch scRNA-seq datasets and you need standardized scores (silhouette, batch entropy, F1, accuracy) plus statistical significance.

## When NOT to use

- Data have no reference cell-type labels — cell-type silhouette, macro F1, and accuracy cannot be computed without ground-truth annotations
- Data have no batch or condition annotation — batch entropy mixing cannot be defined
- You need de novo discovery of unseen cell types rather than comparing methods on shared label sets (the MASI paper notes MASI alone cannot annotate unseen cell types; SCCAF is needed for subtypes)

## Inputs

- Processed scRNA-seq benchmark datasets (e.g., the six mixed-batch datasets from figshare deposit 18866264)
- Predicted cell-type labels from each annotation/integration method (MASI, SVM, scNym, scANVI, Symphony, Scanorama, LIGER, Seurat)
- Reference cell-type labels per cell for supervised metrics (F1, accuracy, silhouette)
- Batch/condition annotation per cell for batch entropy

## Outputs

- Per-method metric table (macro F1, overall accuracy, cell-type silhouette, batch entropy) written to benchmark_results.csv
- Aggregated annotation, integration, and overall performance scores per method
- Paired t-test p-values comparing MASI against each baseline across datasets

## How to apply

Run each competing method (MASI via gene2cell with use_weight=True plus masi.parallel with batch_size=50000 and MACA over-clustering at resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15; SVM classifiers trained on the same cell-type markers; scNym and scArches scANVI with default pipelines; Symphony with variable gene selection, scaling, PCA, and Harmony batch correction; Scanorama, LIGER, Seurat) on the same processed datasets. For each method's output labels and embedding, compute macro F1, overall accuracy, cell-type silhouette, and batch entropy mixing. Aggregate these into three scores (annotation, integration, overall) so that methods can be ranked jointly. Compare MASI against each baseline using paired t-tests across datasets and write all metrics and p-values to benchmark_results.csv; in the MASI paper the threshold for declaring MASI significantly better was p < 0.05 (MASI beat every method except Seurat).

## Related tools

- **MASI** (One benchmarked annotation/integration method whose outputs are scored; produces cell-type scores and labels via masi.gene2cell and masi.parallel) — https://github.com/hayatlab/MASI
- **Scanpy** (Underlying single-cell Python framework in which the benchmark datasets and downstream metric computation operate)
- **Symphony** (Baseline reference-mapping method (variable gene selection, scaling, PCA, Harmony batch correction, query mapping) whose outputs are scored by these metrics) — https://github.com/immunogenomics/symphony
- **scNym** (Baseline semi-supervised adversarial neural-network label-transfer method scored by these metrics with its default pipeline) — https://github.com/calico/scnym

## Evaluation signals

- benchmark_results.csv contains all four metrics (macro F1, accuracy, silhouette, batch entropy) plus p-values for every method and dataset
- Paired t-tests yield p < 0.05 for MASI vs. each baseline (except Seurat), matching the reported finding
- Aggregated annotation, integration, and overall scores are computed for every method so rankings are reproducible
- All methods were run on the identical six processed benchmark datasets and the same cell-type markers were shared between MASI and the SVM classifiers, ensuring fair comparison

## Limitations

- Benchmark conclusions hold for the datasets used in the study; the specific benchmark datasets are not enumerated in the discussion section, limiting generalization
- scNym and scArches were benchmarked only with default pipelines — an optimal outcome for specific data might require customized parameter tuning, so baselines may be under- or over-estimated
- No software version numbers were reported for any tool used in the metric computation pipeline, complicating exact reproduction

## Evidence

- [other] Benchmark metric computation and aggregation steps: [evidence span withheld — anchor status: not_found]
- [other] Statistical significance threshold and outcome: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scope claim: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [discussion] Fair-comparison detail for classifier baselines: "we used the same cell-type markers that were used for MASI to train both linear and non-linear SVM classifiers"
