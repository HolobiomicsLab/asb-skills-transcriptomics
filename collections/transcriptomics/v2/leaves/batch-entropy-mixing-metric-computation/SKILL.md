---
name: batch-entropy-mixing-metric-computation
description: Use when benchmarking multiple scRNA-seq integration/annotation pipelines
  on batch-involved datasets and you need quantitative batch-mixing and cell-type-structure
  metrics plus significance testing between pipelines.
when_to_use_negative:
- The dataset has no batch structure (single-batch experiment) — batch entropy mixing
  is not informative
- The query data contains unseen cell types absent from the reference — MASI alone
  cannot annotate these; combine with SCCAF instead
- Only raw counts are available without log-normalization (scale factor 10,000 + log
  transform) and integrated embeddings — the metrics need annotations and embeddings
  to be meaningful
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: MASI
  role: Generates the 16 pipeline variants of cell-type score matrices (gene2cell
    with use_weight=True) and produces batch annotations (parallel, batch_size=50000)
    that feed the metric computation
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Provides the AnnData objects and preprocessing (e.g., pp.highly_variable_genes
    for HVG feature-set variants) underlying the pipelines being scored
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:3b12016fa06b423c435a47dc44071ca8edd08e547dc9646ac3c6a98ca57b8737
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - Scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# batch-entropy-mixing-metric-computation

## Summary

Computes the batch entropy mixing score and cell-type silhouette score for each of 16 data-processing pipelines across 6 batch-involved scRNA-seq datasets, then runs paired t-tests (p < 0.05) to test whether pipeline #14 significantly outperforms basic pipelines #1–#7 on integration quality.

## When to use

Use when benchmarking multiple scRNA-seq integration/annotation pipelines on batch-involved datasets and you need quantitative batch-mixing and cell-type-structure metrics plus significance testing between pipelines.

## When NOT to use

- The dataset has no batch structure (single-batch experiment) — batch entropy mixing is not informative
- The query data contains unseen cell types absent from the reference — MASI alone cannot annotate these; combine with SCCAF instead
- Only raw counts are available without log-normalization (scale factor 10,000 + log transform) and integrated embeddings — the metrics need annotations and embeddings to be meaningful

## Inputs

- Processed scRNA-seq AnnData objects for six batch-involved datasets from the figshare deposit (figshare.com/articles/dataset/Fast_model-free_integration_and_transfer_learning_via_MASI_for_single-cell_expression_data/18866264)
- Cell-type marker dictionary (cell_markers) derived from reference data via masi.marker_identification(source_data=source)
- Per-cell cell-type annotations and integrated embeddings for each pipeline
- Batch labels for each dataset (required for entropy mixing)

## Outputs

- 16 x 6 summary table of cell-type silhouette scores and batch entropy mixing scores (per pipeline, per dataset)
- Paired t-test statistics (t and p-values) for pipeline #14 vs. each of pipelines #1–#7, with p < 0.05 flagging significant differences

## How to apply

Download the processed datasets for the six batch-involved studies (mouse liver, human pancreas, human hematopoietic, human heart atlas, mouse cortex, mouse brain) from the MASI figshare deposit. For each dataset, generate 16 pipeline variants of feature sets and cell-type score matrices by calling masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), alternating raw and PlinerScore-transformed matrices (0.25 percentile threshold) with marker vs. HVG feature sets. Annotate cells for each pipeline with masi.parallel(scores=scores, labels=labels, batch_size=50000). Using these annotations and the integrated embeddings, compute the cell-type silhouette score and batch entropy mixing score for each pipeline on each dataset, assemble the 16 x 6 summary table, then perform paired t-tests comparing pipeline #14 against pipelines #1–#7 and append the t and p-value statistics to the table.

## Related tools

- **MASI** (Generates the 16 pipeline variants of cell-type score matrices (gene2cell with use_weight=True) and produces batch annotations (parallel, batch_size=50000) that feed the metric computation) — https://github.com/hayatlab/MASI
- **Scanpy** (Provides the AnnData objects and preprocessing (e.g., pp.highly_variable_genes for HVG feature-set variants) underlying the pipelines being scored)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- Output table contains silhouette and batch entropy values for all 16 pipelines across all 6 datasets (no missing cells in the 16 x 6 matrix)
- Paired t-test statistics (t and p-values) are present for pipeline #14 versus each of pipelines #1–#7
- Reported ordering matches the finding that pipeline #14 has substantially better integration scores than pipelines #1–#7 with p < 0.05
- Pipeline #14 shows no significant degradation relative to pipelines #1–#7 on the reported metrics
- Metrics are computed from the MASI annotations and integrated embeddings, not raw gene expression

## Limitations

- Benchmark conclusions are drawn only from 'the datasets used in this study'; the specific benchmark datasets are not fully enumerated in the section text
- Competing methods (scNym, scArches) were benchmarked with default pipelines only; no tuned parameters were reported, which may affect comparability
- No software version numbers are reported for any of the tools used (Scanpy, scikit-learn, MASI, scNym, scArches, Symphony, Seurat, LIGER, Scanorama, Harmony)
- MASI annotation accuracy depends on reference data quality; the choice and resolution of the reference are critical to downstream analysis

## Evidence

- [abstract] Benchmark finding: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [methods] Processed data deposit: "Processed data can be found at https://figshare.com/articles/dataset/Fast_model-free_integration_and_transfer_learning_via_MASI_for_single-cell_expression_data/18866264."
- [methods] gene2cell API: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] Scope of comparison results: "our results show that the performance of MASI is comparable or even superior to other tested methods based on the datasets used in this study."
- [readme] MASI mechanism: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
