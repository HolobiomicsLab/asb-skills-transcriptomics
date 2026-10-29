---
name: single-cell-clustering-metric-evaluation
description: Use when translated or imputed single-cell profiles (e.g. scRNA↔scATAC
  cross-modality predictions) must be scored for preservation of cellular heterogeneity
  against ground-truth cell-type labels.
when_to_use_negative:
- Inputs are unpaired or unlabeled single-modality data with no ground-truth cell-type
  annotations, since label-based concordance metrics cannot be computed
- Evaluating quantitative per-feature accuracy of translated profiles (e.g. gene/peak
  expression correlation) rather than preservation of cellular heterogeneity — use
  feature-level regression metrics instead
edam_operation: http://edamontology.org/operation_0342
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Provides calculate_cluster.calculate_cluster_index to cluster translated profiles
    and compute AMI, ARI, HOM, and NMI against ground-truth cell labels, plus draw_cluster.draw_tsne
    for visualization
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline cross-modality translator whose predicted profiles are scored with
    the same clustering metrics on identical folds
- name: Polarbear
  role: Baseline cross-modality translator whose predicted profiles are scored with
    the same clustering metrics on identical folds
- name: JAMIE
  role: Baseline joint-VAE translator whose predicted profiles are scored with the
    same clustering metrics on identical folds
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:7d2a9734dadd3d3b88cf804b7fefdb81a9feeddf14580c483b3ef6abd275e2a2
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
  edam_operation: http://edamontology.org/operation_0342
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# single-cell-clustering-metric-evaluation

## Summary

Evaluates single-cell cross-modality translations by clustering translated profiles (Leiden) and comparing cluster assignments against ground-truth cell labels using four concordance metrics — AMI, ARI, HOM, and NMI — so that translation quality is scored by how well cellular heterogeneity is preserved.

## When to use

Use when translated or imputed single-cell profiles (e.g. scRNA↔scATAC cross-modality predictions) must be scored for preservation of cellular heterogeneity against ground-truth cell-type labels.

## When NOT to use

- Inputs are unpaired or unlabeled single-modality data with no ground-truth cell-type annotations, since label-based concordance metrics cannot be computed
- Evaluating quantitative per-feature accuracy of translated profiles (e.g. gene/peak expression correlation) rather than preservation of cellular heterogeneity — use feature-level regression metrics instead

## Inputs

- Translated single-cell profiles (predicted scATAC-seq peak matrices or predicted scRNA-seq gene expression matrices from cross-modality translation)
- Ground-truth cell-type labels for the held-out cells
- Fold/dataset metadata identifying the cross-validation partition (e.g. 7 datasets x 5 folds x 2 directions)

## Outputs

- Per-fold clustering concordance metrics: AMI, ARI, HOM, NMI for each method, dataset, fold, and translation direction
- Aggregated benchmark table (e.g. benchmark_clustering_metrics.csv)
- Statistical test results from one-sided paired Wilcoxon signed-rank tests comparing methods over paired per-fold metrics
- Visualization-ready metric distributions for method-comparison plots (e.g. Fig. 2b/c)

## How to apply

After obtaining translated profiles from each method (e.g. scButterfly's test_model output, or BABEL/Polarbear/JAMIE predictions), cluster the translated profiles (Leiden clustering is used by scButterfly's augmentation machinery) and compare cluster assignments with ground-truth cell labels using scButterfly's calculate_cluster.calculate_cluster_index, which computes AMI, ARI, HOM, and NMI. Apply this per fold when data are partitioned (e.g. the paper's five-fold cross-validation by cell across seven paired scRNA-seq/scATAC-seq datasets: BMMC, MB, CL, MCC, MK, MDS, PBMC), in both translation directions (RNA→ATAC and ATAC→RNA). Aggregate fold-level metrics across datasets, folds, and directions (n=140 evaluations in the paper) and run one-sided paired Wilcoxon signed-rank tests over the paired per-fold metric comparisons to assess whether one method significantly outperforms another. Higher AMI/ARI/HOM/NMI values indicate better preservation of cellular heterogeneity in the translated data.

## Related tools

- **scButterfly** (Provides calculate_cluster.calculate_cluster_index to cluster translated profiles and compute AMI, ARI, HOM, and NMI against ground-truth cell labels, plus draw_cluster.draw_tsne for visualization) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline cross-modality translator whose predicted profiles are scored with the same clustering metrics on identical folds)
- **Polarbear** (Baseline cross-modality translator whose predicted profiles are scored with the same clustering metrics on identical folds)
- **JAMIE** (Baseline joint-VAE translator whose predicted profiles are scored with the same clustering metrics on identical folds)

## Examples

```
from scButterfly.calculate_cluster import calculate_cluster_index; ami, ari, hom, nmi = calculate_cluster_index(cell_type, clustering_labels)
```

## Evaluation signals

- All four metrics (AMI, ARI, HOM, NMI) are finite and in the expected range (ARI/NMI/HOM in [0,1]; AMI can be ≤1 with 1 = perfect agreement)
- Per-fold metric counts are complete: 7 datasets x 5 folds x 2 translation directions = 140 paired evaluations per method
- Wilcoxon signed-rank tests are one-sided and paired over per-fold metrics, and scButterfly variants should show significant advantages over BABEL, Polarbear, and JAMIE
- Aggregated CSV (benchmark_clustering_metrics.csv) has one row per dataset/fold/direction/method with all four metric columns populated

## Limitations

- Requires ground-truth cell-type labels for held-out cells; cannot assess heterogeneity preservation on unlabeled data
- Clustering-based metrics depend on the clustering resolution/algorithm (Leiden) and may not capture cell-to-cell variation missed by discrete cluster assignments
- Conclusions depend on the five-fold cross-validation design by cell; other partitioning schemes may change metric variability

## Evidence

- [other] Cluster the translated profiles and compute AMI, ARI, HOM, and NMI against ground-truth cell labels with calculate_cluster.calculate_cluster_index.: [evidence span withheld — anchor status: not_found]
- [results] scButterfly-B achieved better performance than the baseline methods in both translation directions while efficiently characterizing cellular heterogeneity, with significant advantages confirmed by one-sided paired Wilcoxon signed-rank tests.: [evidence span withheld — anchor status: not_found]
- [readme] cluster labels get with Leiden algorithm and MultiVI, a single-cell multi-omics data joint analysis method in Python packages scvi-tools: [evidence span withheld — anchor status: external_doc]
- [methods] draw_tsne from draw_cluster used to visualize clustering of translated profiles.: "calculate_cluster.calculate_cluster_index"
