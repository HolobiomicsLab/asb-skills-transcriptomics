---
name: adjusted-rand-index-evaluation
description: Use when comparing cluster assignments (e.g., from K-means, hierarchical,
  K-medoids, SC3, or Seurat on GEM vs CNDM) against known cell-type or tissue-origin
  labels and a single external-validity score is needed, such as reproducing ARI =
  0.73 on Tabula Muris.
when_to_use_negative:
- No ground-truth cell type or tissue labels are available — ARI is an external validation
  index and cannot be computed against unknown labels
- Evaluating trajectory or pseudotime ordering — ARI measures discrete cluster agreement,
  not differentiation direction or potency (use NFE-based analysis instead)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Constructs the CCSNs and the CNDM whose clusterings are scored by ARI; CNDM-based
    clustering must beat GEM-based clustering on ARI
  repo: https://github.com/LinLi-0909/c-CSN
- name: Seurat
  role: Clusters the dimension-reduced normalized Tabula Muris CNDM; its cluster labels
    are scored against tissue origins to reproduce ARI = 0.73
- name: SC3
  role: One of the clustering methods whose GEM vs CNDM ARI values are compared
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:990d5ef4a531b5b3593e4aeea7e5e064f2abf293be0249c55199d2416b8ff52f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - SC3
  - Seurat
  - c-CSN
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# adjusted-rand-index-evaluation

## Summary

Evaluates scRNA-seq cell-type clustering quality with the adjusted Rand index (ARI) against gold-standard cell labels, as used in the c-CSN paper to show that clustering on the c-CSN-derived conditional network degree matrix (CNDM) outperforms clustering on the original gene expression matrix (GEM) across K-means, hierarchical clustering, K-medoids, SC3, and Seurat.

## When to use

Use when comparing cluster assignments (e.g., from K-means, hierarchical, K-medoids, SC3, or Seurat on GEM vs CNDM) against known cell-type or tissue-origin labels and a single external-validity score is needed, such as reproducing ARI = 0.73 on Tabula Muris.

## When NOT to use

- No ground-truth cell type or tissue labels are available — ARI is an external validation index and cannot be computed against unknown labels
- Evaluating trajectory or pseudotime ordering — ARI measures discrete cluster agreement, not differentiation direction or potency (use NFE-based analysis instead)

## Inputs

- Predicted cluster label vector per cell from a clustering method applied to GEM or CNDM
- Gold-standard cell type label vector (e.g., published cell types for Buettner, Kolodziejczyk, Chu-time; tissue origins bladder/trachea/spleen for Tabula Muris droplet1)
- Number of clusters (published cell type counts per dataset)
- Normalized CNDM (m genes x n cells) and GEM matrices

## Outputs

- Scalar ARI value per method per dataset per matrix type (GEM vs CNDM)
- GEM-vs-CNDM ARI comparison table mirroring Table 1 (datasets: Buettner, Kolodziejczyk, Chu-time)
- Tabula Muris Seurat ARI (~0.73) validated against tissue-origin labels

## How to apply

Run each clustering method (K-means, hierarchical clustering, K-medoids, SC3, SIMLR, or Seurat) on both the normalized gene expression matrix (GEM) and the m x n conditional network degree matrix (CNDM) computed from the CCSNs, using the published cell-type counts as the number of clusters. Compute the ARI between the predicted cluster assignment vector and the gold-standard cell type labels (e.g., with sklearn.metrics.adjusted_rand_score). Assemble ARI values per method per dataset (Buettner GSE65525, Kolodziejczyk GSE74547, Chu-time GSE75748, Tabula Muris GSE109774) into a GEM-vs-CNDM comparison table mirroring Table 1. The rationale is that ARI is chance-corrected, so it fairly credits the higher ARI that CNDM-derived degree data deliver over raw expression data. On Tabula Muris droplet1, the Seurat cluster labels for the three tissue origins (bladder, trachea, spleen) are scored to reproduce ARI = 0.73.

## Related tools

- **c-CSN** (Constructs the CCSNs and the CNDM whose clusterings are scored by ARI; CNDM-based clustering must beat GEM-based clustering on ARI) — https://github.com/LinLi-0909/c-CSN
- **Seurat** (Clusters the dimension-reduced normalized Tabula Muris CNDM; its cluster labels are scored against tissue origins to reproduce ARI = 0.73)
- **SC3** (One of the clustering methods whose GEM vs CNDM ARI values are compared)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(cell_type_labels, cluster_assignments)
```

## Evaluation signals

- For every method-dataset pair in the comparison table, the CNDM ARI is strictly greater than the corresponding GEM ARI (paper's claim: CNDM performs obviously better than GEM on all datasets)
- Tabula Muris droplet1 Seurat ARI reproduces 0.73 within a tolerance of roughly 0.65-0.80
- t-SNE map of the Seurat-clustered Tabula Muris CNDM shows clear separation of the three tissue groups (bladder, trachea, spleen)
- ARI values fall within the valid adjusted-Rand range (approximately -1 to 1, with 1.0 for perfect label agreement)
- Number of clusters supplied to each method matches the published cell type counts for the dataset

## Limitations

- ARI comparison covers only the evaluated datasets (Kim, Buettner, Kolodziejczyk, Chu-time, Tabula Muris); performance on other datasets is not established
- Constructing the CNDM carries a G-fold computational cost increase (one CCSN per cell), and no parallel computation scheme is provided despite this cost
- c-CSN/CNDM is not causal — directions of gene associations cannot be obtained, limiting interpretation of degree-matrix clusters

## Evidence

- [methods] CNDM outperforms GEM: "CNDM performs obviously better than GEM on all datasets"
- [methods] Tabula Muris dataset: "The Tabula Muris droplet1 dataset [47] comprising more than 20,000 cells from three tissues (bladder, trachea, and spleen)"
- [methods] Clustering methods applied: "several traditional clustering methods such as K-means, Hierarchical clustering analysis, and K-medoids were applied to clustering analysis"
- [abstract] Applicability of existing methods: "Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
