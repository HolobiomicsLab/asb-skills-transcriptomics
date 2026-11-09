---
name: spearman-distance-k-medoids-clustering
description: Use when you have simulated (or real) per-feature expression/accessibility
  profiles across multiple cell types and need to verify that features with shared
  simulated regulatory programs group into coherent profile clusters.
when_to_use_negative:
- The data are raw per-cell counts that have not yet been aggregated to per-cell-type
  averages — the profile-level clustering assumes per-cell-type mean profiles
- Single-cell simulation output containing layers other than scRNA-seq and scATAC-seq
  — MOSim's sc_mosim currently supports only those two modalities
- You need clustering of temporal bulk profiles with time-course structure rather
  than across-cell-type profile structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated scRNA-seq and scATAC-seq count matrices (via sc_mosim,
    seeded with pbmc.multiome) whose per-cell-type profiles are clustered and validated
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language in which aggregation, log(x+1) transformation, Spearman distance
    computation, k-medoids (PAM), Silhouette selection, and PCA are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:394b1d4c82564ef7c379fd068970f0f6ad1e2f1bddc9747693b7b9b4be80b181
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spearman-distance-k-medoids-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters feature expression profiles across cell types using Spearman's distance (1 − Spearman correlation) and partition-around-medoids (k-medoids) to validate that simulated multi-omics data reproduce the co-expression/co-accessibility patterns imposed by sc_mosim.

## When to use

Use when you have simulated (or real) per-feature expression/accessibility profiles across multiple cell types and need to verify that features with shared simulated regulatory programs group into coherent profile clusters.

## When NOT to use

- The data are raw per-cell counts that have not yet been aggregated to per-cell-type averages — the profile-level clustering assumes per-cell-type mean profiles
- Single-cell simulation output containing layers other than scRNA-seq and scATAC-seq — MOSim's sc_mosim currently supports only those two modalities
- You need clustering of temporal bulk profiles with time-course structure rather than across-cell-type profile structure

## Inputs

- Simulated or measured per-feature count matrix (scRNA-seq genes / scATAC-seq peaks) with cells labeled by cell type
- Cell-type labels for aggregating counts to per-cell-type averages
- True simulated feature labels (cell type / DE group / co-expression cluster) for validation

## Outputs

- K-medoids cluster assignments for each simulated feature profile
- Spearman distance matrix between feature profiles across cell types
- Silhouette-based optimal cluster number (k)
- PCA plot of log-transformed per-cell-type mean profiles
- Fraction of profiles assigned to an incorrect cluster (e.g. ~3.3% in the reported run)

## How to apply

First aggregate the simulated scRNA-seq/scATAC-seq count matrix to the average count per cell type and apply a log(x+1) transformation. Compute Spearman's distance between feature profiles across cell types as 1 − Spearman's correlation, then cluster the profiles with partition around medoids (k-medoids, e.g. k=10 in the MOSim pbmc.multiome validation). Select the optimal number of clusters by maximizing the Silhouette coefficient while minimizing intra-cluster variability, then run PCA on the log-transformed profiles to confirm clustering aligns with the simulation settings (cell type and DE-group structure). Validation compares cluster assignments against the true simulated labels — in the article's pbmcMultiome run only 3.3% of simulated profiles were misassigned.

## Related tools

- **MOSim** (Generates the simulated scRNA-seq and scATAC-seq count matrices (via sc_mosim, seeded with pbmc.multiome) whose per-cell-type profiles are clustered and validated) — github.com/ConesaLab/MOSim
- **R** (Language in which aggregation, log(x+1) transformation, Spearman distance computation, k-medoids (PAM), Silhouette selection, and PCA are performed)

## Examples

```
library(cluster); D <- 1 - cor(t(logProfiles), method="spearman"); pam_fit <- pam(D, diss=TRUE, k=10); mean(!pam_fit$clustering==trueLabels)
```

## Evaluation signals

- Most feature profiles adhere to the expected mean cluster profile; incorrect-assignment rate reproduces the reported ~3.3%
- Optimal k chosen by combined maximization of Silhouette coefficient and minimization of intra-cluster variability is consistent with the number of simulated co-expression groups (k=10 in the pbmcMultiome example)
- PCA on the log-transformed profiles visually confirms cluster separation aligning with the simulation settings
- Aggregated profiles are log-transformed as log(x+1) before distance computation
- Distance metric is Spearman's distance (1 − Spearman correlation), not Euclidean

## Limitations

- MOSim's single-cell simulation is restricted to scRNA-seq and scATAC-seq, so the clustering validation covers only those two modalities
- MOSim cannot simulate GRNs with spatial resolution, so profiles lack spatial context
- Not applicable to omics layers like proteomics and metabolomics, which may influence gene regulation in more complex or uncertain ways

## Evidence

- [results] Validation outcome from source card: [evidence span withheld — anchor status: not_found]
