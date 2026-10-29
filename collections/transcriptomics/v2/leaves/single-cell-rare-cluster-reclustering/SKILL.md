---
name: single-cell-rare-cluster-reclustering
description: Use when a Seurat object contains a broadly-defined cluster suspected
  to be heterogeneous (low median silhouette scores) and rare subpopulations of very
  low abundance may be masked by standard unsupervised clustering.
when_to_use_negative:
- Clusters already show high, uniform silhouette scores indicating homogeneity — reclustering
  would over-split and manufacture spurious subclusters
- The goal is determining the optimal global cluster number for a whole dataset rather
  than resolving rare cells within an already-defined broad cluster (SCISSORS explicitly
  targets low-abundance subgroups, not optimal-k selection)
- Input is bulk RNA-seq or a pre-computed feature/summary table rather than cell-level
  scRNA-seq count data
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCISSORS
  role: 'Core framework: silhouette-score-guided identification of heterogeneous clusters
    and iterative semi-supervised reclustering to expose rare cell types'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell pipeline wrapping normalization, HVG selection, PCA,
    UMAP, and initial clustering that SCISSORS extends
- name: cluster
  role: R package providing the silhouette function used by ComputeSilhouetteScores
    on cosine distances
- name: CONICSmat
  role: 'Post-reclustering validation: infers aberrant CNVs from the count matrix
    to confirm malignant identity of rare subclusters'
  repo: https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- name: decoderr
  role: 'Post-reclustering validation: computes basal-like pathway weights and bcRatio
    to classify tumor subcluster identity'
  repo: https://github.com/laurapeng/decoderr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:a7a30331ed8567fda79dcb54365d273f582005b66e53b6d1f92c2e62e2ce58c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - CONICSmat
  - R
  - SCISSORS
  - Seurat
  - cluster
  - decoderr
  - dplyr
  - ggplot2
  - magrittr
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-rare-cluster-reclustering

## Summary

Semi-supervised reclustering of broad scRNA-seq clusters with SCISSORS to resolve rare cell subpopulations (e.g. a 148-cell basal-like PDAC tumor subcluster missed by standard Louvain clustering), using silhouette scoring to flag heterogeneous clusters before iterative subclustering.

## When to use

Use when a Seurat object contains a broadly-defined cluster suspected to be heterogeneous (low median silhouette scores) and rare subpopulations of very low abundance may be masked by standard unsupervised clustering.

## When NOT to use

- Clusters already show high, uniform silhouette scores indicating homogeneity — reclustering would over-split and manufacture spurious subclusters
- The goal is determining the optimal global cluster number for a whole dataset rather than resolving rare cells within an already-defined broad cluster (SCISSORS explicitly targets low-abundance subgroups, not optimal-k selection)
- Input is bulk RNA-seq or a pre-computed feature/summary table rather than cell-level scRNA-seq count data

## Inputs

- Seurat object with raw scRNA-seq counts (e.g. Elyada et al. 2019 human PDAC dataset or PBMC3k from SeuratData)
- Cluster labels from initial unsupervised Louvain clustering
- Gene signature sets for validation (e.g. Moffitt basal-like and classical signatures, DECODER pathways)

## Outputs

- Reclustered Seurat object with rare subcluster assignments
- Per-cell silhouette score data frame per broad cluster
- Rare subcluster identity calls with supporting signature scores (VAM Moffitt scores, DECODER bcRatio, CONICSmat CNV profile)

## How to apply

First run a standard Seurat workflow (NormalizeData, FindVariableFeatures with vst, ScaleData, RunPCA) and initial Louvain clustering, then compute per-cell silhouette scores on cosine distance of PCA embeddings with ComputeSilhouetteScores(avg = FALSE) to identify heterogeneous candidate clusters. Prepare the target cells with PrepareData(n.HVG = 4000, n.PC = 15, initial.resolution = 0.4, random.seed = 629) so the low initial resolution prevents rare subgroups from being merged into dominant populations. Apply SCISSORS's multi-step semi-supervised reclustering routine to the heterogeneous cluster — in the Elyada PDAC application, epithelial Clusters 5 and 9 were combined and reclustered to expose a 148-cell (0.59% of dataset) basal-like tumor subcluster. Decision points: recluster only clusters whose silhouette distribution shows low medians, and set the random.seed for reproducibility. Validate recovered subclusters biologically, e.g. with CONICSmat CNV calling (malignant identity), VAM scoring of Moffitt basal-like vs classical signatures, and DECODER basal-like pathway weights/bcRatio with GATA6 depletion.

## Related tools

- **SCISSORS** (Core framework: silhouette-score-guided identification of heterogeneous clusters and iterative semi-supervised reclustering to expose rare cell types) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell pipeline wrapping normalization, HVG selection, PCA, UMAP, and initial clustering that SCISSORS extends)
- **cluster** (R package providing the silhouette function used by ComputeSilhouetteScores on cosine distances)
- **CONICSmat** (Post-reclustering validation: infers aberrant CNVs from the count matrix to confirm malignant identity of rare subclusters) — https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- **decoderr** (Post-reclustering validation: computes basal-like pathway weights and bcRatio to classify tumor subcluster identity) — https://github.com/laurapeng/decoderr

## Examples

```
library(SCISSORS); seu <- PrepareData(seu_epithelial, n.HVG = 4000, n.PC = 15, initial.resolution = 0.4, random.seed = 629); sil_df <- ComputeSilhouetteScores(seu, avg = FALSE)
```

## Evaluation signals

- Silhouette distribution per broad cluster: candidate clusters for reclustering should show lower median silhouette scores than homogeneous siblings
- Recovered rare subcluster should be reproducible under a fixed random.seed (e.g. 629 in the PDAC application)
- In the PDAC case study, the basal-like subcluster must contain 148 cells (0.59% of the dataset) — a concrete size invariant
- Biological validation: significant enrichment of Moffitt basal-like genes with absence of classical genes (VAM), high DECODER basal-like weights and bcRatio, GATA6 depletion, and strong CNV aberrations (CONICSmat)
- Clustering reproducibility: small parameter changes can greatly alter results, so verify reclustering stability across seeds/resolutions before accepting a rare subcluster

## Limitations

- Small differences in analysis and parameter choice can greatly alter clustering results and which cell types are identified
- Existing unsupervised methods struggle with extremely low-abundance cells due to their subtle contribution to global clustering — results for rare groups depend on semi-supervised refinement
- Reclustering a truly homogeneous cluster risks fragmenting noise into pseudo-subclusters; silhouette screening is the guard against this

## Evidence

- [abstract] silhouette-guided reclustering rationale: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] parameter sensitivity caveat: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [abstract] low-abundance challenge motivating the method: "Existing methods largely focus on determining the optimal number of robust clusters, which can be problematic for identifying cells of extremely low abundance"
- [readme] PrepareData invocation with concrete parameters: [evidence span withheld — anchor status: external_doc]
- [methods] per-cell silhouette computation step: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
