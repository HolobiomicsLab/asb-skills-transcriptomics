---
name: brain-cell-type-annotation-from-markers
description: Use when clusters from scCAN (or similar unsupervised clustering) lack
  true cell type labels and a marker-gene reference table (e.g. PanglaoDB brain cell
  types) is available to assign each cluster a known cell type.
when_to_use_negative:
- The dataset already has trusted per-cell type annotations, so validation against
  marker references would be redundant.
- You need to determine the number of clusters or perform the clustering itself —
  use scCAN's eigengap/spectral clustering pipeline first; this skill only annotates
  pre-existing scCAN cluster assignments.
- No curated marker reference (e.g. PanglaoDB) exists for the tissue; Jaccard overlap
  against unavailable marker sets cannot be computed.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: scCAN
  role: Produces the unsupervised cluster assignments (19 clusters) for the Brain
    1.3M dataset whose marker-based validation and cell-type annotation this skill
    performs
  repo: https://cran.r-project.org/package=scCAN
- name: PanglaoDB
  role: Source of the reference marker gene table for 24 brain cell types used as
    ground truth for Jaccard-overlap annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:67699cc675dbc21d710f0773f9a15a206dd46620c5279d2213249358967d7458
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PanglaoDB
  - scCAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# brain-cell-type-annotation-from-markers

## Summary

Biologically validates unsupervised scCAN clusters on the Brain 1.3M dataset (GSE93421) by matching per-cluster differentially expressed genes against PanglaoDB reference marker sets for 24 neural cell types, yielding a 19x24 Jaccard probability matrix and a cluster-to-cell-type mapping.

## When to use

Use when clusters from scCAN (or similar unsupervised clustering) lack true cell type labels and a marker-gene reference table (e.g. PanglaoDB brain cell types) is available to assign each cluster a known cell type.

## When NOT to use

- The dataset already has trusted per-cell type annotations, so validation against marker references would be redundant.
- You need to determine the number of clusters or perform the clustering itself — use scCAN's eigengap/spectral clustering pipeline first; this skill only annotates pre-existing scCAN cluster assignments.
- No curated marker reference (e.g. PanglaoDB) exists for the tissue; Jaccard overlap against unavailable marker sets cannot be computed.

## Inputs

- 1.3M_markers.rds — marker expression matrix for the Brain 1.3M (GSE93421, mouse brain, 1,300,774 cells, 10X Genomics) dataset
- scCAN cluster assignments for the 19 clusters of the Brain 1.3M dataset
- PanglaoDB brain marker table: reference marker genes for 24 biologically validated neural cell types

## Outputs

- 19x24 cluster-by-cell-type Jaccard probability matrix (CSV)
- Cluster-to-cell-type mapping table assigning exactly one highest-probability cell type label to each of the 19 clusters (CSV)

## How to apply

Load the deposited 1.3M marker expression matrix (1.3M_markers.rds) together with the scCAN cluster assignments for the 19 Brain 1.3M clusters. Run get_cluster_markers on the marker expression matrix to derive per-cluster differentially expressed genes, then filter with curate_markers using significance thresholds p <= 0.001 and logFC > 1.5. Load the PanglaoDB brain marker table listing reference genes for 24 brain cell types, and run calculate_celltype_prob to compute Jaccard-overlap probabilities between each cluster's curated marker set and each cell type's marker set. Assign each cluster the cell type with the highest probability (e.g. cluster 1 -> Interneurons) and write both the probability matrix and the mapping table to CSV.

## Related tools

- **scCAN** (Produces the unsupervised cluster assignments (19 clusters) for the Brain 1.3M dataset whose marker-based validation and cell-type annotation this skill performs) — https://cran.r-project.org/package=scCAN
- **PanglaoDB** (Source of the reference marker gene table for 24 brain cell types used as ground truth for Jaccard-overlap annotation)

## Examples

```
library(scCAN); cluster_markers <- get_cluster_markers(1.3M_markers.rds); curated <- curate_markers(cluster_markers, p = 0.001, logFC = 1.5); prob_matrix <- calculate_celltype_prob(curated, panglao_brain_markers)
```

## Evaluation signals

- Output probability matrix has exactly 19 rows (clusters) and 24 columns (cell types).
- Each of the 19 clusters is assigned exactly one cell type label (the argmax of its probability row).
- Curated per-cluster marker lists respect the thresholds p <= 0.001 and logFC > 1.5.
- Jaccard probabilities lie in [0, 1]; a low maximum probability for many clusters suggests poor marker overlap and unreliable annotation.
- Mapping matches expected biology (e.g. cluster 1 -> Interneurons) consistent with the paper's example output.

## Limitations

- The validation assumes the scCAN clustering (and the eigengap-derived 19 clusters) is correct; the approach maps clusters to known types but cannot detect mis-clustering.
- Restricted to the 24 brain cell types in the PanglaoDB reference table — clusters representing cell types absent from the reference may receive a forced, potentially wrong, highest-probability label.
- Jaccard overlap is sensitive to the curation thresholds (p <= 0.001, logFC > 1.5); different cutoffs change the marker sets and hence the probability matrix.

## Evidence

- [other] Marker genes for 24 biologically validated neural cell types retrieved from the PanglaoDB database were used as ground truth to validate the scCAN clustering result on the 1.3M dataset.: [evidence span withheld — anchor status: not_found]
- [other] curate_markers thresholds: [evidence span withheld — anchor status: not_found]
- [other] Jaccard probability matrix computation: [evidence span withheld — anchor status: not_found]
- [readme] scCAN package description: "scCAN is a R software package that can perform unsupervised clustering for single-cell RNA sequencing (scRNA-seq) data."
