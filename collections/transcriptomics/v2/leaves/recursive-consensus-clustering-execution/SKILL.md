---
name: recursive-consensus-clustering-execution
description: Use when clustering transcriptomic datasets (e.g. TCGA pan-cancer, Ivy
  GAP, Neftel scRNA-seq) with an unknown number of clusters, and you need automatic
  k selection plus novel subtype identification.
when_to_use_negative:
- When the number of clusters is already known and a single fixed-k clustering (e.g.
  plain k-means or hierarchical) suffices, since RCC's value is automatic recursive
  k selection
- When analysis time is critical on very large datasets, since RCC's computational
  time is a stated limitation and is tied to the consensus clustering with k-means
  base algorithm
- When manual, domain-expert-driven cluster selection against known annotations is
  required by the analysis protocol
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Core R package providing RCC_clus and downstream annotation/visualization
    functions for recursive consensus clustering
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base clustering algorithm used recursively by RCC_clus to divide the data
    (k-means base)
- name: mclust
  role: Referenced popular clustering method used as comparison baseline
- name: pforeach
  role: Enables parallel execution of RCC's multiple CCP processes (eight processes,
    100 repeats each)
  repo: https://github.com/hoxo-m/pforeach
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:2271848ecd3ff80836e6a49e19ffa271eeeb26fc6de32a069bbe63f9a3c81508
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - RCC (Recursive Consensus Clustering)
  - RCC_clus
  - Recursive Consensus Clustering (RCC)
  - clusterAnnotation
  - clusterAttr
  - geneMarkers
  - mclust
  - pforeach
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# recursive-consensus-clustering-execution

## Summary

Run RCC (Recursive Consensus Clustering) via RCC_clus(x, config_file) to recursively apply ConsensusClusterPlus (k-means base) and automatically select the optimal number of clusters for novel subtype discovery in bulk or single-cell transcriptomic data.

## When to use

Use when clustering transcriptomic datasets (e.g. TCGA pan-cancer, Ivy GAP, Neftel scRNA-seq) with an unknown number of clusters, and you need automatic k selection plus novel subtype identification.

## When NOT to use

- When the number of clusters is already known and a single fixed-k clustering (e.g. plain k-means or hierarchical) suffices, since RCC's value is automatic recursive k selection
- When analysis time is critical on very large datasets, since RCC's computational time is a stated limitation and is tied to the consensus clustering with k-means base algorithm
- When manual, domain-expert-driven cluster selection against known annotations is required by the analysis protocol

## Inputs

- Expression matrix (exprsFile) for a bulk (e.g. Ivy GAP, GTEx, TCGA pan-cancer) or single-cell (e.g. GSE131928 Neftel, GSE57249 Biase) transcriptomic dataset
- RCC config_file specifying base algorithm (ConsensusClusterPlus k-means), repeats, pItem/pFeature subsampling fractions, and parallel process count
- Sample info/attribute file with sample annotations (used downstream by clusterAttr, clusterAnnotation, ssgsea, survival)

## Outputs

- Recursive consensus cluster assignments per sample/cell, including per-level partitions (e.g. Biase ICM vs TE split at level 2)
- Final number of clusters identified automatically per dataset
- Intuitive output plots visualizing recursive clustering results (e.g. Fig 2A style bar plots, Fig 9C concordance plots)

## How to apply

Install the package from github.com/MSCTR/RecursiveConsensusClustering along with dependencies (clv, fields, matrixStats, data.table, cluster, clue, circlize, gdata, scran, ComplexHeatmap, pforeach). Format the expression matrix as the exprsFile input and invoke RCC_clus(x, config_file), which recursively applies ConsensusClusterPlus with a k-means base algorithm to determine the optimal number of clusters without manual selection. For reproducibility/benchmark runs, record the final cluster counts (e.g. 66 for Ivy GAP, 144 for Human tissue, 124 for TCGA pan-cancer, 6 for Biase, 13 for Pollen, 43 for Darmanis, 80 for Neftel) and compute ARI against reference annotations, expecting values in the 0.5–1 range across repeat runs. To trade off speed, run eight parallel CCP processes with variant pItem/pFeature fractions and 100 repeats each, which the paper reports gives higher ARI and shorter wall-clock time than a single process with 800 repetitions. Post-process with clusterAnnotation, clusterAttr, ssgsea, geneMarkers, survival, and trackingPlot to interpret clusters.

## Related tools

- **RCC (Recursive Consensus Clustering)** (Core R package providing RCC_clus and downstream annotation/visualization functions for recursive consensus clustering) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base clustering algorithm used recursively by RCC_clus to divide the data (k-means base))
- **mclust** (Referenced popular clustering method used as comparison baseline)
- **pforeach** (Enables parallel execution of RCC's multiple CCP processes (eight processes, 100 repeats each)) — https://github.com/hoxo-m/pforeach

## Examples

```
devtools::install_github("MSCTR/RecursiveConsensusClustering"); RCC_clus(x, config_file)
```

## Evaluation signals

- Reproduced cluster counts on benchmark datasets equal the reported values: 66 (Ivy GAP), 144 (Human tissue), 124 (TCGA pan-cancer), 6 (Biase), 13 (Pollen), 43 (Darmanis), 80 (Neftel)
- ARI between repeat runs (or against reference annotations) falls within the reported 0.5–1 range
- Parallel configuration (8 processes, variant pItem/pFeature, 100 repeats) yields higher ARI and lower wall-clock time than a single process with 800 repeats (Table S2)
- Biase level-2 partition reproduces the ICM vs trophectoderm split across 1000 runs, where tSNE+k-means and SC3 fail to find these subclusters
- TCGA SKCM resolves into four melanoma subtypes with distinct marker genes and survival profiles

## Limitations

- Computational time is one of the algorithm's limitations and is dependent on the underlying base algorithm (consensus clustering with k-means)
- Analysis of large integrated datasets with millions of cells is the next step and not yet demonstrated; combining with scanorama integration and geometric sketching subsampling is suggested
- Base algorithm is currently fixed to consensus clustering with k-means

## Evidence

- [intro] RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data.: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [methods] RCC_clus signature: "RCC_clus(x, config_file)"
- [discussion] One of the limitations of our algorithm is computational time.: "One of the limitations of our algorithm is computational time."
- [discussion] computational time tied to base algorithm: "This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [readme] installation commands from README: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
