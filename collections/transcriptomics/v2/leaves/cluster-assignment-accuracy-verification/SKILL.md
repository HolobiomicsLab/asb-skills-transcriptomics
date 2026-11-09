---
name: cluster-assignment-accuracy-verification
description: Use when you have simulated single-cell count data with known per-cell-type/DE-group
  ground-truth labels and need to verify that feature profiles adhere to expected
  mean cluster profiles before benchmarking downstream methods.
when_to_use_negative:
- Data are raw single-cell counts that should stay at per-cell resolution — this skill
  operates on per-cell-type aggregated average profiles
- Single-cell layers other than scRNA-seq and scATAC-seq are involved, since sc_mosim
  currently simulates only those two modalities
- The goal is validating bulk temporal patterns, where the paper instead used k-means
  clustering of simulated feature profiles
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: MOSim
  role: R package whose sc_mosim function generates the simulated scRNA-seq/scATAC-seq
    count data and ground-truth DE labels being validated
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language for aggregation, log(x+1) transformation, Spearman distance computation,
    k-medoids clustering, and PCA
- name: SeuratData
  role: Provides the pbmc.multiome 10x Genomics seed dataset used to parameterize
    the simulation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:ef320c57b7a9a168428d4723bd8a2d50f549ff01c8b7f28648b30b1f3353d29a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Cluster-assignment accuracy verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that simulated single-cell multi-omics feature profiles (scRNA-seq and scATAC-seq from MOSim's sc_mosim) reproduce the intended co-expression/DE-group structure by clustering across-cell-type profiles with Spearman's distance and k-medoids (k=10) and measuring the rate of incorrect cluster assignment (~3.3% expected).

## When to use

Use when you have simulated single-cell count data with known per-cell-type/DE-group ground-truth labels and need to verify that feature profiles adhere to expected mean cluster profiles before benchmarking downstream methods.

## When NOT to use

- Data are raw single-cell counts that should stay at per-cell resolution — this skill operates on per-cell-type aggregated average profiles
- Single-cell layers other than scRNA-seq and scATAC-seq are involved, since sc_mosim currently simulates only those two modalities
- The goal is validating bulk temporal patterns, where the paper instead used k-means clustering of simulated feature profiles

## Inputs

- MOSim sc_mosim simulated count matrices for scRNA-seq and scATAC-seq (six cell types, two groups, three replicates, pbmcMultiome seed)
- Ground-truth simulated cell-type / DE-group labels per feature

## Outputs

- K-medoids cluster assignments for feature profiles across cell types (k=10)
- Incorrect cluster-assignment rate (target ≈3.3%)
- PCA plot of log(x+1)-transformed profiles confirming alignment with simulation settings
- Silhouette coefficient / intra-cluster variability values supporting choice of k

## How to apply

Aggregate the simulated count matrix to the average count per cell type and apply a log(x+1) transformation. Compute Spearman's distance (1 − Spearman's correlation) between feature profiles across cell types and cluster them with partition-around-medoids (k-medoids, k=10); select the optimal k by maximizing Silhouette's coefficient while minimizing intra-cluster variability. Run PCA on the log-transformed profiles to visually confirm the clustering aligns with the simulation settings. Finally, compare k-medoids cluster assignments against the true simulated cell-type/DE-group labels; a correct reproduction of the paper's validation shows only ~3.3% of simulated profiles assigned to an incorrect cluster.

## Related tools

- **MOSim** (R package whose sc_mosim function generates the simulated scRNA-seq/scATAC-seq count data and ground-truth DE labels being validated) — github.com/ConesaLab/MOSim
- **R** (Language for aggregation, log(x+1) transformation, Spearman distance computation, k-medoids clustering, and PCA)
- **SeuratData** (Provides the pbmc.multiome 10x Genomics seed dataset used to parameterize the simulation)

## Examples

```
avg <- log(rowMeanByCellType(counts) + 1); d <- as.dist(1 - cor(t(avg), method='spearman')); pam(d, k=10)
```

## Evaluation signals

- Incorrect cluster-assignment rate against true simulated labels reproduces the reported ~3.3%
- Silhouette coefficient is maximized while intra-cluster variability is minimized at the chosen k
- PCA of log(x+1)-transformed profiles shows clusters aligned with the simulation settings (cell type / DE group)
- Most features fall into the expected mean cluster profiles for both scRNA-seq and scATAC-seq layers

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, so this validation cannot cover other single-cell modalities
- Validation relies on aggregated per-cell-type profiles, so cell-to-cell heterogeneity within a type is not assessed
- MOSim cannot simulate GRNs with spatial resolution and does not cover proteomics or metabolomics layers

## Evidence
