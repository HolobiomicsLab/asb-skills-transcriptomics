---
name: paired-ttest-method-comparison
description: 'Use when two or more methods are evaluated on the same datasets and
  you need a statistically grounded ranking: compute per-dataset metrics for every
  method, then run paired t-tests of the focal method against each baseline.'
when_to_use_negative:
- Methods are evaluated on different datasets or data splits, so observations are
  not paired and a paired t-test is invalid
- Only a single dataset is available, leaving too few paired observations for a meaningful
  t-test
- The question is per-cell label accuracy on one dataset rather than cross-dataset
  method ranking
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Focal method under benchmarking; label transfer via gene2cell and parallel
    batch annotation
  repo: https://github.com/hayatlab/MASI
- name: SVM (linear and non-linear)
  role: Baseline classifiers trained on the same cell-type markers used by MASI
- name: scNym
  role: Baseline annotation method run with its default pipeline
  repo: https://github.com/calico/scnym
- name: scArches scANVI
  role: Baseline annotation method run with its default pipeline
  repo: https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- name: Symphony
  role: Baseline integration/annotation method; reference built with variable gene
    selection, scaling, PCA, and Harmony correction
  repo: https://github.com/immunogenomics/symphony
- name: Scanorama
  role: Baseline integration method run on the same datasets
- name: LIGER
  role: Baseline integration method run on the same datasets
- name: Seurat
  role: Baseline integration/annotation method; only method not significantly outperformed
    by MASI
- name: MACA
  role: Over-clustering engine used within MASI label transfer at resolutions 3, 5,
    7 and neighborhood sizes 5, 10, 15
- name: Scanpy
  role: Used to compute cell-type silhouette and batch entropy mixing metrics on annotations
    and integrated embeddings
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:12ad88b52c3342bfc1456cc4dc1cacf61477235e03eefb1b7516f0157514e095
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - Harmony
  - LIGER
  - MACA
  - MASI
  - SVM (linear and non-linear)
  - Scanorama
  - Scanpy
  - Seurat
  - Symphony
  - scArches
  - scArches scANVI
  - scNym
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# paired-ttest-method-comparison

## Summary

Benchmark multiple methods on the same set of datasets and test performance differences with paired t-tests, as done in the MASI paper where MASI was compared against SVM, scNym, scArches scANVI, Symphony, Scanorama, LIGER, and Seurat using macro F1, accuracy, silhouette, and batch entropy metrics.

## When to use

Use when two or more methods are evaluated on the same datasets and you need a statistically grounded ranking: compute per-dataset metrics for every method, then run paired t-tests of the focal method against each baseline.

## When NOT to use

- Methods are evaluated on different datasets or data splits, so observations are not paired and a paired t-test is invalid
- Only a single dataset is available, leaving too few paired observations for a meaningful t-test
- The question is per-cell label accuracy on one dataset rather than cross-dataset method ranking

## Inputs

- Processed scRNA-seq benchmark datasets (six mixed-batch studies) in AnnData/h5ad format from figshare deposit 18866264
- Per-method per-dataset metric values (macro F1, overall accuracy, cell-type silhouette, batch entropy mixing)
- Aggregated annotation, integration, and overall performance scores per method

## Outputs

- benchmark_results.csv containing metrics and paired t-test p-values for each method comparison
- 16 x 6 (or methods x datasets) summary table of silhouette and batch entropy values
- Paired t-test statistics (t and p-values) for focal method vs. each baseline

## How to apply

Run every method (e.g., MASI, linear/non-linear SVM, scNym, scArches scANVI, Symphony, Scanorama, LIGER, Seurat) on the identical benchmark datasets — here six mixed-batch scRNA-seq studies from figshare deposit 18866264 — using each tool's stated pipeline (MASI via gene2cell with MACA resolutions 3/5/7 and neighborhood sizes 5/10/15; Symphony via reference build with HVG selection, scaling, PCA, Harmony; scNym/scArches with default pipelines). For each method-dataset pair compute macro F1, overall accuracy, cell-type silhouette score, and batch entropy mixing score, then aggregate into annotation, integration, and overall performance scores. Because all methods see the same datasets, treat datasets as paired samples and perform paired t-tests comparing the focal method against each baseline, writing metrics and p-values to benchmark_results.csv; in the MASI study, differences with p < 0.05 were called significant.

## Related tools

- **MASI** (Focal method under benchmarking; label transfer via gene2cell and parallel batch annotation) — https://github.com/hayatlab/MASI
- **SVM (linear and non-linear)** (Baseline classifiers trained on the same cell-type markers used by MASI)
- **scNym** (Baseline annotation method run with its default pipeline) — https://github.com/calico/scnym
- **scArches scANVI** (Baseline annotation method run with its default pipeline) — https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- **Symphony** (Baseline integration/annotation method; reference built with variable gene selection, scaling, PCA, and Harmony correction) — https://github.com/immunogenomics/symphony
- **Scanorama** (Baseline integration method run on the same datasets)
- **LIGER** (Baseline integration method run on the same datasets)
- **Seurat** (Baseline integration/annotation method; only method not significantly outperformed by MASI)
- **MACA** (Over-clustering engine used within MASI label transfer at resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15)
- **Scanpy** (Used to compute cell-type silhouette and batch entropy mixing metrics on annotations and integrated embeddings)

## Examples

```
from scipy import stats; t, p = stats.ttest_rel(masi_scores, baseline_scores); benchmark_results.append({'baseline': name, 't': t, 'p': p})
```

## Evaluation signals

- benchmark_results.csv contains metrics for every method-dataset pair and paired t-test t- and p-values for the focal method vs. each baseline
- All methods were run on the identical six benchmark datasets, satisfying the pairing assumption of the t-test
- Reported significance calls use the stated threshold (p-values < 0.05) and identify which baselines are (and are not) significantly outperformed — e.g., MASI significantly above all except Seurat
- Metrics span the four dimensions actually compared: macro F1, overall accuracy, cell-type silhouette, and batch entropy mixing
- Per-dataset metric ordering matches the aggregated performance scores (annotation, integration, overall) reported

## Limitations

- Baselines were run only with default pipelines; optimal per-dataset results might require customized parameter tuning, which was not done for the benchmark
- Conclusions are limited to the datasets used in the study; generalization to other tissues or batches is not established
- No software version numbers were reported for any benchmarked tool, complicating exact reproduction

## Evidence

- [results] Across the six benchmark datasets, MASI achieved significantly higher overall performance than all other tested methods: [evidence span withheld — anchor status: not_found]
- [discussion] we used default pipelines of both methods for all data involved in this study: "an optimal outcome for a specific data might require customized parameter tuning. However, for benchmarking, we used default pipelines of both methods for all data"
- [abstract] We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [discussion] our results show that the performance of MASI is comparable or even superior to other tested methods based on the datasets used: "our results show that the performance of MASI is comparable or even superior to other tested methods based on the datasets used in this study."
- [other] Compute the cell-type silhouette score and batch entropy mixing score for each pipeline on each dataset: [evidence span withheld — anchor status: not_found]
