---
name: label-transfer-across-scrna-seq-methods
description: Use when multiple scRNA-seq methods must be compared head-to-head for
  cell-type annotation and integration, using standardized metrics and paired statistical
  tests across datasets.
when_to_use_negative:
- The query data contains cell types unseen in the reference — MASI alone cannot annotate
  those; combine with SCCAF or use a different approach
- Only one method needs to be applied for production annotation rather than a multi-method
  benchmark with statistical comparison
- The reference data quality or resolution does not match the query — annotation accuracy
  depends critically on the choice and resolution of the reference
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Primary model-free label-transfer method under evaluation; uses gene2cell
    scoring and MACA over-clustering
  repo: https://github.com/hayatlab/MASI
- name: MACA
  role: Performs over-clustering of query cells for label transfer, with clustering
    resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15
- name: scNym
  role: Semi-supervised adversarial neural network baseline for cell-type classification,
    run with default pipelines
  repo: https://github.com/calico/scnym
- name: Symphony
  role: Reference-mapping baseline; builds a reference with variable gene selection,
    scaling, PCA, and Harmony batch correction, then maps queries
  repo: https://github.com/immunogenomics/symphony
- name: scArches scANVI
  role: Deep-learning annotation baseline run with the default scanvi surgery pipeline
  repo: https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- name: Scanpy
  role: Provides preprocessing (pp.highly_variable_genes) and AnnData handling across
    the benchmark
- name: Harmony
  role: Batch correction used within the Symphony reference-building step
- name: Seurat
  role: Integration baseline (R); also required as an MASI dependency for marker rank
    aggregation
- name: LIGER
  role: Integration baseline compared against MASI
- name: Scanorama
  role: Integration baseline compared against MASI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:024295c62b3481fa2b6beeb48278d6ddfcce068e5d0a175ebfbc2f5c43a0d686
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
  - scArches scANVI
  - scNym
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# label-transfer-across-scrna-seq-methods

## Summary

Benchmark cell-type label transfer and data integration across model-free (MASI) and model-based baselines (SVM, scNym, scArches scANVI, Symphony, Scanorama, LIGER, Seurat) using macro F1, accuracy, cell-type silhouette, batch entropy, and paired t-tests.

## When to use

Use when multiple scRNA-seq methods must be compared head-to-head for cell-type annotation and integration, using standardized metrics and paired statistical tests across datasets.

## When NOT to use

- The query data contains cell types unseen in the reference — MASI alone cannot annotate those; combine with SCCAF or use a different approach
- Only one method needs to be applied for production annotation rather than a multi-method benchmark with statistical comparison
- The reference data quality or resolution does not match the query — annotation accuracy depends critically on the choice and resolution of the reference

## Inputs

- Processed scRNA-seq datasets (six mixed-batch benchmark datasets from figshare deposit 18866264)
- Reference Anndata object with cell-type labels in the 'cell_type' obs column
- Cell-type marker gene dictionaries per cell type

## Outputs

- Per-method cell-type annotations for each dataset
- Metrics: macro F1, overall accuracy, cell-type silhouette, batch entropy mixing
- Aggregated annotation, integration, and overall performance scores
- Paired t-test p-values comparing MASI vs each baseline (written to benchmark_results.csv)

## How to apply

Load the six processed scRNA-seq benchmark datasets from the figshare deposit 18866264 and log-normalize raw counts (divide by total counts per cell, multiply by 10,000, then log-transform). Run MASI via masi.marker_identification on the reference Anndata (labels in 'cell_type'), then masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) followed by masi.parallel(scores=scores, labels=labels, batch_size=50000); MACA over-clustering sweeps resolutions 3, 5, 7 with neighborhood sizes 5, 10, 15, and large data are split into N batches (one MACA run per CPU core). Train linear and non-linear SVM classifiers on the same cell-type markers used by MASI, run scNym and scArches scANVI with default pipelines, and run Symphony (reference build with variable gene selection, scaling, PCA, Harmony correction, then query mapping), Scanorama, LIGER, and Seurat on the same data. For MASI, skip highly variable gene selection since only identified marker genes are used, and suppress marker expression below the 0.25 PlinerScore percentile threshold to zero. Aggregate metrics into annotation, integration, and overall performance scores, then perform paired t-tests comparing MASI against each baseline across datasets.

## Related tools

- **MASI** (Primary model-free label-transfer method under evaluation; uses gene2cell scoring and MACA over-clustering) — https://github.com/hayatlab/MASI
- **MACA** (Performs over-clustering of query cells for label transfer, with clustering resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15)
- **scNym** (Semi-supervised adversarial neural network baseline for cell-type classification, run with default pipelines) — https://github.com/calico/scnym
- **Symphony** (Reference-mapping baseline; builds a reference with variable gene selection, scaling, PCA, and Harmony batch correction, then maps queries) — https://github.com/immunogenomics/symphony
- **scArches scANVI** (Deep-learning annotation baseline run with the default scanvi surgery pipeline) — https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- **Scanpy** (Provides preprocessing (pp.highly_variable_genes) and AnnData handling across the benchmark)
- **Harmony** (Batch correction used within the Symphony reference-building step)
- **Seurat** (Integration baseline (R); also required as an MASI dependency for marker rank aggregation)
- **LIGER** (Integration baseline compared against MASI)
- **Scanorama** (Integration baseline compared against MASI)

## Examples

```
cell_markers = masi.marker_identification(source_data=source); scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- benchmark_results.csv contains macro F1, accuracy, cell-type silhouette, batch entropy, and paired t-test p-values for every method and all six datasets
- Paired t-tests yield p < 0.05 for MASI vs all baselines except Seurat, per the reported finding
- MASI's rank is among the top methods for both integration and annotation aggregate scores
- MASI gene2cell returns a (scores, labels) pair whose annotation can be reproduced with use_weight=True and batch_size=50000
- Preprocessing matches LogNormalization (scale factor 10,000 followed by log-transformation) before any label transfer

## Limitations

- MASI annotation accuracy depends on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not present in the reference; it must be combined with SCCAF to identify subtypes within major cell types
- scNym and scArches benchmarks used default pipelines; customized parameter tuning might yield better baseline results, and no tuned parameters were reported
- No software version numbers were reported for any of the tools (Scanpy, scikit-learn, MASI, scNym, scArches, Symphony, Seurat, LIGER, Scanorama, Harmony)
- The PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not stated, and the number of batches N for splitting large data is unspecified

## Evidence

- [methods] MASI benchmark protocol: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] Parallel batch annotation: "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [discussion] MACA clustering sweep: "These include clustering resolution 3, 5, 7 with 5, 10, 15 as neighborhood sizes to over-cluster cells"
- [discussion] Skipping HVG selection: "we skipped the step of calling highly variable genes, because only the identified marker genes were used for integrative annotation."
- [discussion] SVM baseline on same markers: "we used the same cell-type markers that were used for MASI to train both linear and non-linear SVM classifiers"
- [abstract] Benchmark outcome finding: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [discussion] Unseen cell types limitation: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [discussion] Log normalization detail: [evidence span withheld — anchor status: not_found]
