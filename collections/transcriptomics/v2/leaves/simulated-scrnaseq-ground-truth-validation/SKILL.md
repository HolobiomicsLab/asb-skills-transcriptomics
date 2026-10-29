---
name: simulated-scrnaseq-ground-truth-validation
description: Use when evaluating or comparing clustering methods on scRNA-seq data
  where simulated ground-truth cluster labels exist, especially to test whether a
  method resolves rare subpopulations placed close together.
when_to_use_negative:
- The dataset has no ground-truth (or simulated) cluster labels, so ARI/NMI against
  true clusters cannot be computed — use biological marker-based annotation instead
- The goal is rare-cell discovery on real data rather than method benchmarking — run
  the clustering method (e.g. SCISSORS) directly rather than this validation harness
- Input is already a cluster-label table needing only annotation, not an end-to-end
  clustering comparison
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_4085
tools:
- name: SCISSORS
  role: 'Method under test: iterative silhouette-based semi-supervised reclustering
    wrapped around Seurat, evaluated for rare-cluster separation'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Baseline Louvain clustering at resolutions 0.5–2.0 for the resolution sweep
    comparison
- name: cluster
  role: cluster::silhouette routine used to compute per-cell silhouette scores on
    PCA-space distances
- name: ggplot2
  role: Violin plots of per-method silhouette distributions and resolution-sweep plots
- name: ggsignif
  role: Adds wilcox.test significance bars to the silhouette violin plots
- name: dplyr
  role: Data wrangling of metrics and label tables during the comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:4791e0efe4f95d11e7b778056cb61f0b070713f44e480d16529703157791d7bc
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_4085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# simulated-scrnaseq-ground-truth-validation

## Summary

Benchmark scRNA-seq clustering methods against a simulated dataset with known ground-truth labels (here a 996-cell, 15,270-gene simulated pancreas set with 7 true clusters including two rare closely placed clusters), scoring external agreement (ARI, NMI) and internal geometry (per-cell silhouette on PCA-space distances) to quantify rare-cluster separation.

## When to use

Use when evaluating or comparing clustering methods on scRNA-seq data where simulated ground-truth cluster labels exist, especially to test whether a method resolves rare subpopulations placed close together.

## When NOT to use

- The dataset has no ground-truth (or simulated) cluster labels, so ARI/NMI against true clusters cannot be computed — use biological marker-based annotation instead
- The goal is rare-cell discovery on real data rather than method benchmarking — run the clustering method (e.g. SCISSORS) directly rather than this validation harness
- Input is already a cluster-label table needing only annotation, not an end-to-end clustering comparison

## Inputs

- Simulated scRNA-seq count/processed matrix with 996 cells × 15,270 genes (simulated pancreas dataset)
- Ground-truth vector of 7 cluster labels for all 996 cells
- PCA-space embedding of the preprocessed data (dims used for neighbor/silhouette computation)
- Shared t-SNE embedding coordinates for cross-method visualization

## Outputs

- Per-method metrics table of ARI, NMI, and aggregate silhouette score
- Per-cell silhouette score distributions per method
- Violin plot comparing silhouette distributions per method with wilcox.test significance bars (ggsignif)
- t-SNE panels showing each method's cluster assignments
- Louvain resolution-sweep plot (ARI, NMI, silhouette vs. resolution 0.5–2.0)
- Saved metrics table and PDF figures

## How to apply

Load the simulated scRNA-seq matrix (996 cells × 15,270 genes) together with its 7 ground-truth cluster labels, and run each candidate method on the preprocessed data — including the method under test (SCISSORS reclustering) and baselines such as Seurat Louvain clustering swept across resolutions 0.5, 1.0, 1.5, and 2.0. Compute per-cell silhouette scores with the cluster::silhouette routine on PCA-space distances, aggregating them per method, and compute external agreement (ARI and NMI) of each method's labels against the ground truth. Visualize per-method cluster assignments on a shared t-SNE embedding and build a ggplot2 comparison of silhouette distributions as violin plots with wilcox.test significance bars via ggsignif. For baseline tuning, plot a Louvain resolution sweep showing ARI, NMI, and silhouette as functions of resolution from 0.5 to 2.0. The rationale is that silhouette scoring estimates cluster heterogeneity, so per-cell silhouette distributions plus ARI/NMI expose whether rare, closely placed clusters are merged by a given method.

## Related tools

- **SCISSORS** (Method under test: iterative silhouette-based semi-supervised reclustering wrapped around Seurat, evaluated for rare-cluster separation) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Baseline Louvain clustering at resolutions 0.5–2.0 for the resolution sweep comparison)
- **cluster** (cluster::silhouette routine used to compute per-cell silhouette scores on PCA-space distances)
- **ggplot2** (Violin plots of per-method silhouette distributions and resolution-sweep plots)
- **ggsignif** (Adds wilcox.test significance bars to the silhouette violin plots)
- **dplyr** (Data wrangling of metrics and label tables during the comparison)

## Examples

```
sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)
```

## Evaluation signals

- Reference result: SCISSORS attains ARI = 0.987, NMI = 0.985, and silhouette = 0.693 on the simulated pancreas set and is the only method among eight to separate the two rare clusters
- Resolution-sweep invariant: the lowest Louvain resolution (0.5) yields the highest ARI/NMI/silhouette — increasing resolution must not improve clustering
- Metrics table must contain one ARI/NMI/silhouette row per candidate method computed against the 7 ground-truth labels
- Silhouette values must be computed per cell (avg = FALSE) on PCA-space distances before aggregation
- All figures are exported as PDFs and the metrics table is saved alongside them

## Limitations

- Silhouette-based reclustering performance is sensitive to analysis and parameter choice: 'even small differences in analysis and parameter choosing can greatly alter clustering results'
- Validation relies on simulated data; the simulated rare clusters are two closely placed clusters, so conclusions about other rarity geometries are not directly supported
- No discussion-section text was available, so no additional reproducibility gaps or limitations could be extracted from the paper's discussion

## Evidence

- [abstract] SCISSORS silhouette scoring for heterogeneity: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Parameter sensitivity caveat: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [readme] Package purpose from README: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine"
- [methods] Per-cell silhouette computation call: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
