---
name: normalized-entropy-label-agreement-computation
description: Use when you need to quantify how stable or reproducible two sets of
  cell cluster labels are — e.g., comparing labels from a perturbed matrix (masked
  AS values, sub-sampled cells) against reference labels.
when_to_use_negative:
- The goal is differential AS analysis rather than clustering agreement — SCASL itself,
  or tools like Leafcutter/BRIE, are the wrong layer for quantifying label concordance.
- The perturbation changes the number of clusters or QC thresholds relative to the
  reference, making entropy comparisons ill-posed.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCASL
  role: Produces the AS-based cluster labels (spectral clustering on imputed AS probability
    matrix) whose stability against reference labels is quantified by normalized entropy
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Junction-read-based strategy that SCASL mirrors for de novo identification
    of AS events feeding the probability matrix
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:38ca985ab26f4eb0ed4f48771914b8540ce3054ae68eeab00d8ff51774d6838b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# normalized-entropy-label-agreement-computation

## Summary

Computes normalized entropy between a reference cell clustering and clusterings obtained under perturbations (forced dropout, cell sub-sampling) to quantify cluster stability, used in SCASL to show AS-profile clusters are more robust than Seurat-style gene-expression clusters.

## When to use

Use when you need to quantify how stable or reproducible two sets of cell cluster labels are — e.g., comparing labels from a perturbed matrix (masked AS values, sub-sampled cells) against reference labels.

## When NOT to use

- The goal is differential AS analysis rather than clustering agreement — SCASL itself, or tools like Leafcutter/BRIE, are the wrong layer for quantifying label concordance.
- The perturbation changes the number of clusters or QC thresholds relative to the reference, making entropy comparisons ill-posed.

## Inputs

- Reference cluster label vector (e.g., SCASL preds from the full TNBC-2 AS probability matrix)
- Perturbed cluster label vectors (from dropout-masked and 100%/75%/50% cell sub-sampled matrices)
- Comparator cluster label vectors from gene-expression-based clustering
- SCASL normalized AS probability matrix (normalized_matrix, rows=sites, columns=cells)

## Outputs

- Per-condition cluster label files for each dropout/sub-sampling condition
- stability_entropy.csv containing normalized entropy values per condition for SCASL vs. the gene-expression comparator

## How to apply

Establish reference labels by running SCASL (spectral clustering with iterative weighted KNN imputation, parameters `num_iteration` and `knn` in `configs/srr.yaml`) on the unperturbed AS probability matrix from `data/junction` (TNBC-2 demo). Generate perturbed inputs by masking ~50% of AS values (forced dropout) and by randomly retaining 100%, 75%, and 50% of cells, keeping `num_cluster` and QC thresholds (`sites_initial`, `runs_initial`, `sites_quality`, `runs_quality`) fixed. Re-cluster each perturbed matrix, then compute normalized entropy between each perturbed clustering's labels and the reference labels; repeat for a canonical gene-expression clustering comparator. Write per-condition labels plus the SCASL-vs-expression entropy table to stability_entropy.csv.

## Related tools

- **SCASL** (Produces the AS-based cluster labels (spectral clustering on imputed AS probability matrix) whose stability against reference labels is quantified by normalized entropy) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Junction-read-based strategy that SCASL mirrors for de novo identification of AS events feeding the probability matrix) — https://github.com/davidaknowles/leafcutter

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- SCASL's normalized entropy is lower than (i.e., more stable than) the gene-expression comparator's across the 50% dropout and 100%/75%/50% sub-sampling conditions, consistent with the paper's reported robustness result.
- stability_entropy.csv contains one entropy value per perturbation condition for both SCASL and the expression-based comparator.
- `num_cluster` and QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) are identical between reference and perturbed runs, so label counts are comparable.
- Per-condition label files align cell-by-cell with the reference label vector (or restricted to retained cells in sub-sampling conditions).

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts, so even imputed SCASL labels retain residual sensitivity to severe masking.
- Demo files in `data/junction` (TNBC-2) are minimal; due to the randomness of imputation, dimensionality reduction and unsupervised clustering, entropy values may vary between runs.
- Interpretability of entropy differences requires matching cluster numbers; canonical gene-expression clustering may find a different number of clusters, complicating direct comparison.

## Evidence

- [results] Under different cell sub-sampling ratios, the clusters defined by SCASL from single-cell AS profiles are much more robust and stable than those defined by Seurat from gene expression profiles.: [evidence span withheld — anchor status: not_found]
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [readme] cluster_algorithm: "cluster_algorithm: 'spectral' by default. ('spectral', 'agglomerative', 'kmeans'. )"
- [readme] QC thresholds: "you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing"
