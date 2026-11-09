---
name: ari-nmi-benchmark-computation
description: Use when comparing the accuracy or robustness of scRNA-seq clustering
  methods (e.g., scAce vs. Leiden) on data with ground-truth cell-type labels, especially
  across repeated random-cell-subset repeats where median ARI/NMI per method is the
  target metric.
when_to_use_negative:
- The dataset has no ground-truth cell-type labels — ARI and NMI are external agreement
  metrics and cannot be computed
- The goal is clustering-parameter tuning on a single fit rather than cross-method
  robustness benchmarking over repeats
- The output needed is an internal clustering-quality score (e.g., silhouette) rather
  than agreement with reference labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scanpy
  role: Wraps the expression matrix in an AnnData and performs the preprocessing (filter_genes
    min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before
    clustering and scoring
- name: scAce
  role: Clustering method under evaluation; produces cluster labels (adata.obs['scace_cluster'])
    that are scored against ground truth
  repo: https://github.com/sldyns/scAce
- name: Leiden
  role: Baseline clustering method; with de novo initialization it supplies initial
    cluster labels and serves as a comparison point in the ARI/NMI benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:cbc470e2d9577c2f3e8a21be9769c93f480064e184654227a1e8499cbd0adced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ari-nmi-benchmark-computation

## Summary

Computes Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) between each clustering method's predicted labels and ground-truth cell-type labels, tabulated across repeated subsampling runs to quantify clustering accuracy and robustness (as in the scAce benchmark).

## When to use

Use when comparing the accuracy or robustness of scRNA-seq clustering methods (e.g., scAce vs. Leiden) on data with ground-truth cell-type labels, especially across repeated random-cell-subset repeats where median ARI/NMI per method is the target metric.

## When NOT to use

- The dataset has no ground-truth cell-type labels — ARI and NMI are external agreement metrics and cannot be computed
- The goal is clustering-parameter tuning on a single fit rather than cross-method robustness benchmarking over repeats
- The output needed is an internal clustering-quality score (e.g., silhouette) rather than agreement with reference labels

## Inputs

- Simulated or real single-cell gene expression matrix with ground-truth cell-type labels as a scanpy AnnData object
- Predicted cluster label vectors (e.g., adata.obs['scace_cluster']) from each of the ten clustering methods per repeat
- Per-cell count metadata (adata.obs.n_counts) used to record the scale factor

## Outputs

- Full per-repeat ARI/NMI table for every method (e.g., robustness_ari_nmi.csv)
- Per-method median ARI and median NMI across the ten repeats

## How to apply

Load the gene expression matrix with ground-truth labels into a scanpy AnnData, then pre-process following the scAce README: filter_genes with min_cells=3, filter_cells with min_genes=200, normalize_per_cell, record scale_factor (n_counts / median n_counts), log1p, and scale. Run each clustering method — scAce via run_scace (ZINB-VAE pre-training, Leiden-based de novo initialization, adaptive cluster merging) plus baselines such as Leiden — on each repeat (e.g., ten random 95% cell subsets). Compute ARI and NMI between predicted labels and ground truth for every method on every repeat, take the median across repeats per method, and write the per-repeat table and medians (e.g., to robustness_ari_nmi.csv). The median across repeats is used because robustness, not just peak accuracy, is the comparison criterion.

## Related tools

- **scanpy** (Wraps the expression matrix in an AnnData and performs the preprocessing (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before clustering and scoring)
- **scAce** (Clustering method under evaluation; produces cluster labels (adata.obs['scace_cluster']) that are scored against ground truth) — https://github.com/sldyns/scAce
- **Leiden** (Baseline clustering method; with de novo initialization it supplies initial cluster labels and serves as a comparison point in the ARI/NMI benchmark)

## Examples

```
from scace import run_scace; adata = run_scace(adata); ari = adjusted_rand_score(adata.obs['ground_truth'], adata.obs['scace_cluster'])
```

## Evaluation signals

- The output CSV contains one ARI and one NMI value per method per repeat (10 repeats x 10 methods) plus a per-method median
- The median, not the mean, is reported across repeats, consistent with the robustness experiment design
- ARI and NMI are computed against the same ground-truth cell-type labels used to build the simulated data
- Validation check: scAce attains the highest median ARI and highest median NMI among the ten methods, consistent with reported better accuracy and robustness
- All methods were run on the same randomly sampled 95% cell subsets per repeat, so pairwise comparisons share identical cell inputs

## Limitations

- ARI/NMI require reference labels, so this skill only applies to data with known ground truth (e.g., simulated data); real datasets without labels cannot be scored this way
- Robustness conclusions depend on the number of repeats (10) and the subsample size (95% of cells); medians can vary with different repeat schemes
- scAce's clustering-enhancement mode depends on prior clustering results from other methods, so baseline quality can feed back into scAce's labels

## Evidence

- [abstract] we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness: "we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness"
- [readme] With de novo initialization, Leiden is used to obtain initial cluster labels: "With *de novo* initialization, Leiden is used to obtain initial cluster labels"
- [readme] The output adata contains cluster labels in adata.obs['scace_cluster'] and the cell embeddings in adata.obsm['scace_emb']: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
