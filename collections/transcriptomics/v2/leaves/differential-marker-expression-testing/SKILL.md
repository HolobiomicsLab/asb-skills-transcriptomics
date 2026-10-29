---
name: differential-marker-expression-testing
description: Use when clustering results (e.g. 19 scCAN clusters on the Brain 1.3M
  dataset, GSE93421) lack true cell type labels and must be biologically validated
  and mapped to known cell types via marker genes.
when_to_use_negative:
- When the single-cell dataset already has trusted per-cell cell-type labels — supervised
  annotation or direct ARI comparison (as in the scCAN README example using adjustedRandIndex)
  is more appropriate
- When no curated reference marker set exists for the tissue's expected cell types
  — Jaccard overlap against an unrelated marker panel would produce meaningless probabilities
- Before clustering has been performed — this skill validates an existing partition;
  run scCAN (autoencoder + network fusion + spectral clustering) first to obtain cluster
  assignments
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCAN
  role: Provides the cluster assignments and marker expression matrices that get_cluster_markers,
    curate_markers, and calculate_celltype_prob operate on for marker-based biological
    validation
  repo: https://cran.r-project.org/package=scCAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:b0129a4b6cfd2cb05d3f1dc6e220d261611373ea5243d4422f11be56e552c225
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scCAN
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# differential-marker-expression-testing

## Summary

Identifies differentially expressed marker genes for each scCAN cluster and validates clusters biologically by computing Jaccard-overlap probabilities against reference marker gene sets (e.g. 24 PanglaoDB brain cell types), producing a cluster-by-cell-type probability matrix and a one-to-one cluster-to-cell-type mapping.

## When to use

Use when clustering results (e.g. 19 scCAN clusters on the Brain 1.3M dataset, GSE93421) lack true cell type labels and must be biologically validated and mapped to known cell types via marker genes.

## When NOT to use

- When the single-cell dataset already has trusted per-cell cell-type labels — supervised annotation or direct ARI comparison (as in the scCAN README example using adjustedRandIndex) is more appropriate
- When no curated reference marker set exists for the tissue's expected cell types — Jaccard overlap against an unrelated marker panel would produce meaningless probabilities
- Before clustering has been performed — this skill validates an existing partition; run scCAN (autoencoder + network fusion + spectral clustering) first to obtain cluster assignments

## Inputs

- Marker expression matrix for the clustered dataset (e.g. 1.3M_markers.rds, genes x clusters or cells x genes for the 1.3M-cell Brain dataset)
- scCAN cluster assignments (cluster labels for the 19 clusters of the Brain 1.3M dataset)
- Reference cell-type marker gene table from PanglaoDB (marker genes for 24 biologically validated brain cell types)

## Outputs

- Per-cluster curated differentially expressed gene lists (p <= 0.001, logFC > 1.5)
- Cluster-by-cell-type Jaccard probability matrix written to CSV (19 rows x 24 columns for Brain 1.3M vs PanglaoDB brain cell types)
- Cluster-to-cell-type mapping table written to CSV, one assigned label per cluster

## How to apply

Load the deposited marker expression matrix (e.g. 1.3M_markers.rds) together with the scCAN cluster assignments (19 clusters for Brain 1.3M) and run get_cluster_markers to find differentially expressed genes per cluster. Curate each cluster's marker list with curate_markers using significance thresholds p <= 0.001 and logFC > 1.5 to retain only strong markers. Load a reference marker table (e.g. the PanglaoDB brain table listing marker genes for 24 brain cell types) and run calculate_celltype_prob to compute Jaccard-overlap probabilities between each cluster's curated markers and each cell type's markers, yielding a 19x24 probability matrix. Assign each cluster the cell type with the highest probability (e.g. cluster 1 -> Interneurons) and write both the probability matrix and the cluster-to-cell-type mapping table to CSV. The Jaccard overlap is used because it is a simple set-based similarity that directly compares curated marker gene sets against curated reference sets without requiring expression-level calibration across datasets.

## Related tools

- **scCAN** (Provides the cluster assignments and marker expression matrices that get_cluster_markers, curate_markers, and calculate_celltype_prob operate on for marker-based biological validation) — https://cran.r-project.org/package=scCAN

## Examples

```
curated <- curate_markers(get_cluster_markers(markers_1.3M, clusters), p = 0.001, logFC = 1.5); probs <- calculate_celltype_prob(curated, panglao_brain_markers)
```

## Evaluation signals

- Output probability matrix has exactly 19 rows (clusters) and 24 columns (cell types) for the Brain 1.3M / PanglaoDB brain comparison
- Each cluster is assigned exactly one cell type label, taken from the column with the highest Jaccard probability in its row
- Curated marker lists contain only genes passing p <= 0.001 and logFC > 1.5
- Probability matrix values are valid Jaccard proportions in [0, 1], with the assigned label's cell consistent with the row maximum
- Both the probability matrix and the cluster-to-cell-type mapping table are written as CSV files

## Limitations

- Clusters lack true cell type labels, so validation is entirely reference-based and only as good as the PanglaoDB marker sets used as ground truth
- Strict thresholds (p <= 0.001, logFC > 1.5) may discard weaker but biologically meaningful markers, lowering Jaccard overlap for legitimate matches
- The approach assumes the number of clusters (19 for Brain 1.3M, from scCAN's eigengap-based estimate) reasonably corresponds to the reference cell-type space (24 types); mismatches leave some clusters ambiguous

## Evidence

- [methods] Filter the per-cluster marker lists with curate_markers using the significance thresholds p <= 0.001 and logFC > 1.5.: [evidence span withheld — anchor status: not_found]
- [methods] Jaccard probability matrix: [evidence span withheld — anchor status: not_found]
- [results] PanglaoDB ground truth: [evidence span withheld — anchor status: not_found]
- [readme] scCAN README: "scCAN overcomes the excessive noise level from scRNA-seq data by using autoencoders and accurately cluster the cells into correct cell types."
