---
name: silhouette-score-evaluation-of-cell-type-clustering
description: Use when comparing scRNA-seq integration/annotation pipelines (e.g.,
  16 MASI variants of raw vs. PlinerScore matrices, marker vs. HVG features) and you
  need a numeric, statistically testable measure of whether cell-type structure is
  resolved.
when_to_use_negative:
- When the question is annotation correctness against ground-truth labels (accuracy/F1)
  rather than unsupervised cluster geometry — the article benchmarks MASI on integration,
  annotation, and speed as separate axes
- When batch mixing alone is the target and cell-type separation is not scored — batch
  entropy mixing should be interpreted jointly with silhouette, not in isolation
- When the reference data quality or cell-type resolution is not matched to the query
  — MASI annotation is dependent on reference quality, so silhouette scores on poor
  annotations are not interpretable
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Generates the 16 pipeline variants of cell-type score matrices (raw vs. PlinerScore-transformed,
    marker vs. HVG features) and produces the annotations that the silhouette and
    batch entropy scores are computed on
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Provides preprocessing and HVG feature sets for pipeline variants (e.g., pp.highly_variable_genes)
    used as comparison arms in the benchmark
- name: SCCAF
  role: Optional companion for identifying subtypes within major cell types when the
    reference lacks unseen cell types
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:e4af754454fd1bda9181e2b7896971931002efbaa402abdb7153f855c984b3c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# silhouette-score-evaluation-of-cell-type-clustering

## Summary

Quantitatively score how well cell-type annotations form compact, well-separated clusters by computing the cell-type silhouette score (paired with batch entropy mixing) per pipeline per dataset, then test significance with paired t-tests. Used in the MASI benchmark to rank 16 feature/score pipelines across 6 batch-involved scRNA-seq datasets.

## When to use

Use when comparing scRNA-seq integration/annotation pipelines (e.g., 16 MASI variants of raw vs. PlinerScore matrices, marker vs. HVG features) and you need a numeric, statistically testable measure of whether cell-type structure is resolved.

## When NOT to use

- When the question is annotation correctness against ground-truth labels (accuracy/F1) rather than unsupervised cluster geometry — the article benchmarks MASI on integration, annotation, and speed as separate axes
- When batch mixing alone is the target and cell-type separation is not scored — batch entropy mixing should be interpreted jointly with silhouette, not in isolation
- When the reference data quality or cell-type resolution is not matched to the query — MASI annotation is dependent on reference quality, so silhouette scores on poor annotations are not interpretable

## Inputs

- Processed scRNA-seq AnnData objects for six batch-involved studies (mouse liver, human pancreas, human hematopoietic, human heart atlas, mouse cortex, mouse brain) from the figshare deposit (figshare.com/articles/dataset/18866264)
- Per-pipeline cell-type annotation labels from masi.parallel (scores from masi.gene2cell with use_weight=True)
- Integrated embeddings for each pipeline variant

## Outputs

- A 16 x 6 summary table of cell-type silhouette scores (values in [-1, 1])
- A 16 x 6 summary table of batch entropy mixing scores
- Paired t-test statistics (t and p-values) comparing pipeline #14 against pipelines #1–#7

## How to apply

For each dataset (e.g., mouse liver, human pancreas, human hematopoietic, human heart atlas, mouse cortex, mouse brain) and each pipeline variant, first produce cell-type annotations from the cell-type score matrix via masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) followed by masi.parallel(scores=scores, labels=labels, batch_size=50000). Then compute two complementary metrics on each pipeline's annotations and integrated embeddings: the cell-type silhouette score (cluster compactness/separation; bounded in [-1, 1]) and the batch entropy mixing score (batch-effect resolution). Assemble the scores into a 16 x 6 matrix per metric, and assess whether pipeline #14 significantly outperforms the basic pipelines #1–#7 with paired t-tests (significance threshold p-value < 0.05), appending t and p statistics to the summary table. The rationale is that a marker-based method should only be preferred over simpler variants if it shows no significant degradation on both cell-type and batch metrics simultaneously.

## Related tools

- **MASI** (Generates the 16 pipeline variants of cell-type score matrices (raw vs. PlinerScore-transformed, marker vs. HVG features) and produces the annotations that the silhouette and batch entropy scores are computed on) — https://github.com/hayatlab/MASI
- **Scanpy** (Provides preprocessing and HVG feature sets for pipeline variants (e.g., pp.highly_variable_genes) used as comparison arms in the benchmark)
- **SCCAF** (Optional companion for identifying subtypes within major cell types when the reference lacks unseen cell types)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- The output table contains silhouette and batch entropy values for all 16 pipelines across all 6 datasets (complete 16 x 6 grid, no missing cells)
- Paired t-test rows exist for pipeline #14 vs. each of pipelines #1–#7 with both t and p-values; significance flag matches p-value < 0.05
- All silhouette scores fall within the valid range [-1, 1]
- The reported pipeline ordering matches the finding that pipeline #14 shows no significant degradation relative to pipelines #1–#7 on the reported metrics
- Each dataset's scores are computed on that pipeline's own annotations and integrated embedding (paired design preserved for the t-tests)

## Limitations

- Benchmark conclusions hold only for the datasets used in the study; the specific benchmark datasets are not enumerated in the discussion section
- No software version numbers are reported for Scanpy, scikit-learn, MASI, scNym, scArches, Symphony, Seurat, LIGER, Scanorama, or Harmony, limiting exact reproducibility
- MASI annotation accuracy (and hence downstream silhouette scores) depends on the quality, choice, and resolution of the reference data
- MASI alone cannot annotate unseen cell types not present in the reference; combining with SCCAF is required to identify subtypes within major cell types

## Evidence

- [abstract] We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed.: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [methods] gene2cell converts gene expression to cell-type scores used for annotation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] parallel annotates large score matrices in batches of 50000 cells: "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [discussion] Benchmark performance is conditioned on the datasets used in the study: "our results show that the performance of MASI is comparable or even superior to other tested methods based on the datasets used in this study."
- [readme] MASI identifies markers from reference data and transfers labels through MACA, supplying the annotations scored by silhouette/entropy: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
