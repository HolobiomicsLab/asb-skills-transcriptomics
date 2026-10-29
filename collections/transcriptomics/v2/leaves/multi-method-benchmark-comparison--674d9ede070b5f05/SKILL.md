---
name: multi-method-benchmark-comparison
description: Use when comparing a new cross-modality single-cell translation method
  against published baselines on identical data folds, needing per-fold paired statistics
  over AMI, ARI, HOM, and NMI clustering indices.
when_to_use_negative:
- The goal is a single-method evaluation without head-to-head baseline comparison
  on shared folds — a paired benchmark design is unnecessary
- Data are unpaired or from modalities other than scRNA-seq/scATAC-seq (e.g., spatial
  transcriptomics or single-cell Hi-C), which the article names as future directions
  not yet supported
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scButterfly
  role: Candidate method; trained per fold via Butterfly.construct_model/train_model
    as B (no augmentation), T (cell_type_augmentation), and C (MultiVI_augmentation)
    variants, then translated with test_model
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline autoencoder-based cross-modality translator run on identical folds
    for comparison
- name: Polarbear
  role: Baseline VAE-stitching cross-modality translator run on identical folds for
    comparison
- name: JAMIE
  role: Baseline joint-VAE multi-modal translator run on identical folds for comparison
- name: Python
  role: Implementation environment; scButterfly installed on Python 3.9 via pip or
    the GitHub wheel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:17111cefbc348af5ab30aafaa1a1c6b4b624cd86b18856889efd59049ac66604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - Polarbear
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-method-benchmark-comparison

## Summary

A rigorous five-fold cross-validation benchmarking protocol comparing a candidate method (scButterfly variants B/T/C) against established baselines (BABEL, Polarbear, JAMIE) across seven paired scRNA-seq/scATAC-seq datasets, evaluating translation quality by clustering-index metrics and one-sided paired Wilcoxon signed-rank tests.

## When to use

Use when comparing a new cross-modality single-cell translation method against published baselines on identical data folds, needing per-fold paired statistics over AMI, ARI, HOM, and NMI clustering indices.

## When NOT to use

- The goal is a single-method evaluation without head-to-head baseline comparison on shared folds — a paired benchmark design is unnecessary
- Data are unpaired or from modalities other than scRNA-seq/scATAC-seq (e.g., spatial transcriptomics or single-cell Hi-C), which the article names as future directions not yet supported

## Inputs

- Seven paired scRNA-seq/scATAC-seq AnnData objects (BMMC, MB, CL, MCC, MK, MDS, PBMC) as raw count matrices
- Five-fold cell-level train/test/validation ID splits per dataset
- Ground-truth cell-type labels for clustering index computation
- Baseline translator outputs from BABEL, Polarbear, and JAMIE on identical folds

## Outputs

- Per-fold AMI, ARI, HOM, NMI values for each method, direction (R2A and A2R), dataset, and fold (n=140 evaluations)
- benchmark_clustering_metrics.csv with aggregated paired metrics
- Method-comparison visualizations analogous to Fig. 2b/c
- One-sided paired Wilcoxon signed-rank test results establishing significance of scButterfly's advantage

## How to apply

Load the seven paired scRNA-seq/scATAC-seq datasets (BMMC, MB, CL, MCC, MK, MDS, PBMC) via Butterfly.load_data and preprocess with RNA_data_preprocessing and ATAC_data_preprocessing from scButterfly.data_processing. Partition cells into five disjoint folds and, per fold, train scButterfly-B (aug_type=None), scButterfly-T (cell_type_augmentation using labels in adata.obs), and scButterfly-C (MultiVI_augmentation via scvi-tools), then translate held-out cells in both directions with Butterfly.test_model; run BABEL, Polarbear, and JAMIE on the identical folds. Cluster the translated profiles and compute AMI, ARI, HOM, and NMI against ground-truth labels using calculate_cluster.calculate_cluster_index. Aggregate fold-level metrics across 7 datasets x 5 folds x 2 directions (n=140 paired evaluations), visualize method comparisons as in Fig. 2b/c, and assess superiority with one-sided paired Wilcoxon signed-rank tests, writing the aggregated table to benchmark_clustering_metrics.csv.

## Related tools

- **scButterfly** (Candidate method; trained per fold via Butterfly.construct_model/train_model as B (no augmentation), T (cell_type_augmentation), and C (MultiVI_augmentation) variants, then translated with test_model) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline autoencoder-based cross-modality translator run on identical folds for comparison)
- **Polarbear** (Baseline VAE-stitching cross-modality translator run on identical folds for comparison)
- **JAMIE** (Baseline joint-VAE multi-modal translator run on identical folds for comparison)
- **Python** (Implementation environment; scButterfly installed on Python 3.9 via pip or the GitHub wheel)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- All 7 datasets x 5 folds x 2 translation directions yield complete metric entries (n=140), with baselines evaluated on identical cell folds
- Clustering indices (AMI, ARI, HOM, NMI) computed against ground-truth labels via calculate_cluster_index fall within plausible [0,1] ranges
- One-sided paired Wilcoxon signed-rank tests over paired per-fold metrics show significant advantage of scButterfly-B over each baseline in both directions
- Aggregated table benchmark_clustering_metrics.csv is written with per-method, per-dataset, per-fold rows
- scButterfly-T and scButterfly-C variants each show augmentation-derived gains over scButterfly-B as reported in the paper

## Limitations

- Augmentation strategies (scButterfly-T/C) increase training time compared to scButterfly-B
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via adaptive, interpretable model parameters
- Benchmarks rely on paired scRNA-seq/scATAC-seq data; unpaired scenarios and additional modalities (spatial transcriptomics, Hi-C) are not covered by this evaluation protocol

## Evidence

- [other] In five-fold cross-validation by cell across seven datasets, scButterfly-B achieved better performance than the baseline methods in both translation directions while efficiently characterizing cellular heterogeneity: [evidence span withheld — anchor status: not_found]
- [other] significant advantages confirmed by one-sided paired Wilcoxon signed-rank tests: [evidence span withheld — anchor status: not_found]
- [methods] calculate_cluster_index API: "calculate_cluster.calculate_cluster_index"
- [methods] Preprocessing functions for both modalities: "data_processing.RNA_data_preprocessing"
- [readme] Prediction API returning both translation directions: [evidence span withheld — anchor status: external_doc]
- [readme] Augmentation variant choice defining the B/T/C model family: [evidence span withheld — anchor status: external_doc]
