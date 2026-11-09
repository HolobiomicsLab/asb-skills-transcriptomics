---
name: single-cell-rnaseq-clustering-benchmarking
description: Use when a normalized scRNA-seq dataset with known cell type labels is
  available and the goal is to test whether clustering the c-CSN-transformed degree
  matrix (CNDM) outperforms clustering the gene expression matrix (GEM), measured
  by ARI.
when_to_use_negative:
- The dataset has no gold-standard cell type labels, so ARI against reference labels
  cannot be computed.
- The input is already a network degree matrix (e.g., a prior CNDM); transforming
  it again via c-CSN is outside this skill's scope.
- The goal is trajectory/branch inference (Monocle, TSCAN, DPT) rather than discrete
  cell-type clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Constructs the CCSN for each cell and aggregates them into the CNDM used as
    the alternative clustering input
  repo: https://github.com/LinLi-0909/c-CSN
- name: SC3
  role: Consensus clustering method run on both GEM and CNDM with the published cell
    type count as number of clusters
- name: Seurat
  role: Additional clustering method applied to CNDM in the article's clustering analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:902bf1fbbae0c80cd5382eb0b3f98fb20da47f84856d2be9bf3ee38e8ecfe780
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - CIDR
  - R
  - SC3
  - SIMLR
  - SSCC
  - Seurat
  - ZIFA
  - c-CSN
  - dropClust
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rnaseq-clustering-benchmarking

## Summary

Benchmarks standard scRNA-seq clustering methods (K-means, SC3, hierarchical, K-medoids) on the c-CSN-derived conditional network degree matrix (CNDM) versus the original gene expression matrix (GEM), scoring each clustering with adjusted Rand index (ARI) against gold-standard cell type labels.

## When to use

Use when a normalized scRNA-seq dataset with known cell type labels is available and the goal is to test whether clustering the c-CSN-transformed degree matrix (CNDM) outperforms clustering the gene expression matrix (GEM), measured by ARI.

## When NOT to use

- The dataset has no gold-standard cell type labels, so ARI against reference labels cannot be computed.
- The input is already a network degree matrix (e.g., a prior CNDM); transforming it again via c-CSN is outside this skill's scope.
- The goal is trajectory/branch inference (Monocle, TSCAN, DPT) rather than discrete cell-type clustering.

## Inputs

- normalized gene expression matrix (GEM) for a labeled scRNA-seq dataset (e.g., Buettner, Kolodziejczyk, Chu-time)
- gold-standard cell type labels and published cell type counts per dataset
- c-CSN-constructed CCSNs per cell aggregated into the m × n conditional network degree matrix (CNDM)

## Outputs

- ARI table of GEM vs CNDM per clustering method (K-means, hierarchical, K-medoids, SC3) per dataset
- cluster label assignments for each method on each input matrix

## How to apply

Download a labeled scRNA-seq dataset (e.g., Buettner GSE65525, Kolodziejczyk GSE74547, or Chu-time GSE75748) and normalize the gene expression matrix. For each cell, construct conditional cell-specific networks (CCSNs) using c-CSN — a hypothesis test on the normalized conditional statistic against the standard-normal significance level, with conditional genes selected as the top-G largest-degree hub genes from the CSN — then aggregate CCSNs into the m × n CNDM where entry c_ij is the degree of gene-gene interactions. Run K-means, hierarchical clustering, and K-medoids (scikit-learn/scipy) plus SC3 on both the GEM and the CNDM, setting the number of clusters to the published cell type count. Compute adjusted Rand index of every clustering against the gold-standard labels and assemble a GEM-vs-CNDM comparison table per method per dataset. The paper's finding is that CNDM performs obviously better than GEM on all datasets, so CNDM ARI > GEM ARI is the expected direction in every pairwise comparison.

## Related tools

- **c-CSN** (Constructs the CCSN for each cell and aggregates them into the CNDM used as the alternative clustering input) — https://github.com/LinLi-0909/c-CSN
- **SC3** (Consensus clustering method run on both GEM and CNDM with the published cell type count as number of clusters)
- **Seurat** (Additional clustering method applied to CNDM in the article's clustering analysis)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score; pred = KMeans(n_clusters=n_types).fit_predict(CNDM); ari = adjusted_rand_score(cell_type_labels, pred)
```

## Evaluation signals

- CNDM ARI > GEM ARI for each method-dataset pair, matching the paper's claim that 'CNDM performs obviously better than GEM on all datasets'.
- Number of clusters in each run equals the published cell type count for the dataset.
- CNDM dimensions are m × n: m genes (or selected genes) by n cells, one aggregated degree value per gene per cell from G CCSNs.
- ARI values are in [0, 1] and the comparison table mirrors Table 1 structure (methods × datasets × GEM/CNDM).
- Each clustering is computed on the same normalized matrix for GEM and CNDM so that ARI differences isolate the transformation, not preprocessing.

## Limitations

- c-CSN incurs a G-fold computational cost increase (one network per conditional gene), and no parallel computation scheme is provided despite the authors stating it is desired.
- Clustering on the CNDM is only meaningful for methods designed for gene expression matrices that were shown applicable to c-CSN-transformed degree matrices.
- ARI comparison is limited to datasets with trusted gold-standard cell type labels; results on unlabeled data cannot be validated this way.

## Evidence

- [methods] CNDM performs better than GEM in clustering of scRNA-seq data (higher ARI): "CNDM performs obviously better than GEM on all datasets"
- [methods] CNDM construction from CCSNs: "an m × n matrix conditional network degree matrix (CNDM) is obtained"
- [abstract] Applicability of existing methods to degree matrices: "Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
- [intro] Clustering methods named in the article: "clustering methods such as hierarchical clustering, K-means, SNN-Cliq [18], Corr [19], SC3 [20], and SIMLR [21]"
- [abstract] Software availability: "c-CSN is publicly available at https://github.com/LinLi-0909/c-CSN."
