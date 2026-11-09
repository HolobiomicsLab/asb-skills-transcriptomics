---
name: adjusted-rand-index-and-nmi-computation
description: Use when a scRNA-seq dataset has ground-truth cluster labels (e.g. a
  simulated pancreas dataset with 7 true clusters) and you must score predicted cluster
  assignments against them, as in the SCISSORS comparison workflow.
when_to_use_negative:
- The dataset has no ground-truth cluster labels — ARI/NMI require reference assignments;
  use silhouette alone for internal quality.
- The question is subcluster/rare-cell discovery within broad clusters, not benchmarking
  label agreement against known populations.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: One of the eight clustering methods evaluated; its labels are scored by ARI/NMI
    against ground truth
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Provides Louvain clustering at resolutions 0.5–2.0 whose labels are scored,
    wrapping the single-cell pipeline
- name: cluster
  role: cluster::silhouette computes per-cell silhouette scores on PCA-space distances
    as the internal-quality companion to ARI/NMI
- name: ggplot2
  role: Builds the resolution-sweep and silhouette violin-plot comparison figures
- name: ggsignif
  role: Adds wilcox.test significance bars to the silhouette violin plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:f4d674f2953254b387358be0d762b7fe12510eb11dc423f324608268a95cc902
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - magrittr
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Adjusted Rand Index and NMI Computation

## Summary

Computes external clustering agreement metrics — Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) — between predicted scRNA-seq cluster assignments and ground-truth labels, to quantify how well methods such as SCISSORS or Seurat Louvain recover known cell populations.

## When to use

Use when a scRNA-seq dataset has ground-truth cluster labels (e.g. a simulated pancreas dataset with 7 true clusters) and you must score predicted cluster assignments against them, as in the SCISSORS comparison workflow.

## When NOT to use

- The dataset has no ground-truth cluster labels — ARI/NMI require reference assignments; use silhouette alone for internal quality.
- The question is subcluster/rare-cell discovery within broad clusters, not benchmarking label agreement against known populations.

## Inputs

- Simulated scRNA-seq dataset (996 cells × 15,270 genes, preprocessed)
- Ground-truth cluster label vector (7 clusters, including two rare closely placed clusters)
- Predicted cluster label vectors from each clustering method (SCISSORS, Seurat Louvain at resolutions 0.5/1.0/1.5/2.0, plus the other compared methods)
- PCA-space distance matrix for silhouette computation

## Outputs

- Per-method ARI and NMI values against ground truth
- Metrics table combining ARI, NMI, and silhouette score per method
- Resolution-sweep figure (ARI, NMI, silhouette vs. Louvain resolution 0.5–2.0) saved as PDF
- Violin-plot comparison of per-cell silhouette distributions (ggplot2 with ggsignif wilcox.test bars)

## How to apply

Run each candidate clustering method on the preprocessed data — e.g. SCISSORS reclustering and Seurat Louvain at resolutions 0.5, 1.0, 1.5, and 2.0 — to obtain per-cell cluster labels. For each method, compute ARI and NMI of the predicted labels against the 7 ground-truth clusters of the simulated 996-cell pancreas dataset. Pair these external metrics with internal quality by computing per-cell and aggregate silhouette scores with the cluster::silhouette routine on PCA-space distances. Assemble the ARI/NMI/silhouette values into a metrics table, and, for the resolution sweep, plot ARI, NMI, and silhouette as a function of Louvain resolution from 0.5 to 2.0 (here the lowest resolution, 0.5, gave the best ARI/NMI/silhouette, so increasing resolution did not improve clustering). Save the metrics table and figures as PDFs; in this benchmark SCISSORS was expected to reach ARI = 0.987 and NMI = 0.985.

## Related tools

- **SCISSORS** (One of the eight clustering methods evaluated; its labels are scored by ARI/NMI against ground truth) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Provides Louvain clustering at resolutions 0.5–2.0 whose labels are scored, wrapping the single-cell pipeline)
- **cluster** (cluster::silhouette computes per-cell silhouette scores on PCA-space distances as the internal-quality companion to ARI/NMI)
- **ggplot2** (Builds the resolution-sweep and silhouette violin-plot comparison figures)
- **ggsignif** (Adds wilcox.test significance bars to the silhouette violin plots)

## Examples

```
sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE); ari <- mclust::adjustedRandIndex(clusters_k3, truth_labels)
```

## Evaluation signals

- SCISSORS must attain ARI = 0.987 and NMI = 0.985 on the simulated pancreas dataset
- SCISSORS must be the only method that successfully separates the two rare closely placed clusters
- Aggregate silhouette score for SCISSORS should be 0.693 (second highest of the eight methods)
- Resolution sweep: resolution 0.5 should yield the highest ARI/NMI/silhouette, so increasing resolution does not improve clustering
- Metrics table values reproduce exactly when recomputed from saved per-method label vectors

## Limitations

- Even small differences in analysis and parameter choosing can greatly alter clustering results, so ARI/NMI conclusions are sensitive to preprocessing and parameter choice
- External metrics like ARI/NMI depend on the ground-truth labels; on real datasets without trusted references they cannot be computed directly
- No discussion-section limitations were extractable from the provided document

## Evidence

- [other] SCISSORS achieved the highest ARI (0.987), the highest NMI (0.985), and the second highest silhouette score (0.693): [evidence span withheld — anchor status: not_found]
- [other] Compute external agreement metrics (ARI and NMI) of each method's labels against the 7 ground-truth clusters: [evidence span withheld — anchor status: not_found]
- [methods] silhouette via cluster routine on PCA-space distances: "library(cluster)         # clustering tools"
- [abstract] parameter choice alters results: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [abstract] silhouette estimation in SCISSORS: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
