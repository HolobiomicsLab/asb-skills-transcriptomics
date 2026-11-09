---
name: spatial-transcriptomics-spot-clustering
description: Use when you have a spot-level ratio matrix (e.g., RUD APA usage from
  stAPAminer) plus spot coordinates from spatially barcoded ST data (e.g., MOB) and
  need to cluster spots, embed them, and validate clusters against tissue layer annotations.
when_to_use_negative:
- Input is raw sequencing data rather than a quantified poly(A) site matrix — the
  PA-extraction pipeline (umi_tools, STAR, scAPAtrap) must run first.
- Data are image-based ST (MERFISH/seqFISH) without spatial barcodes — the article
  states APA sites can only be identified from spatially barcoded ST data with existing
  tools.
- The goal is gene-level differential APA testing (DEAPA/LSAPA/SVAPA) rather than
  spot-level layer clustering and its validation.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: provides createStAPAminerObject, makeStCluster, findLabels, RenameCluster,
    drawClusterPlot, computeIndex, computePearsonIndex for spot clustering and evaluation
  repo: https://github.com/BMILAB/stAPAminer
- name: Seurat
  role: underlying engine for graph-based clustering, dimensionality reduction, and
    UMAP embedding invoked by makeStCluster
- name: ggplot2
  role: visualization of cluster plots, UMAP, and volcano plots
- name: clusterSim
  role: supporting package for clustering evaluation indices
- name: ClusterR
  role: provides distinctColorPalette used for cluster plot coloring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:394b7983cfa94bf5bf7ddb2f7d98e4dd55af05d42238d5eedf52fb7ebe1d45cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - ClusterR
  - Seurat
  - cluster
  - clusterSim
  - fdm2id
  - ggplot2
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# spatial-transcriptomics-spot-clustering

## Summary

Graph-based clustering of spatial transcriptomics spots on an APA usage (RUD) matrix using stAPAminer/Seurat, followed by UMAP embedding and quantitative evaluation against morphological layer labels via adjusted Rand index and within-layer spot-spot Pearson correlations.

## When to use

Use when you have a spot-level ratio matrix (e.g., RUD APA usage from stAPAminer) plus spot coordinates from spatially barcoded ST data (e.g., MOB) and need to cluster spots, embed them, and validate clusters against tissue layer annotations.

## When NOT to use

- Input is raw sequencing data rather than a quantified poly(A) site matrix — the PA-extraction pipeline (umi_tools, STAR, scAPAtrap) must run first.
- Data are image-based ST (MERFISH/seqFISH) without spatial barcodes — the article states APA sites can only be identified from spatially barcoded ST data with existing tools.
- The goal is gene-level differential APA testing (DEAPA/LSAPA/SVAPA) rather than spot-level layer clustering and its validation.

## Inputs

- poly(A) site matrix / PACdataset of APA counts per spot (movAPA APA.RDA)
- gene-spot count matrix (count.csv)
- spot coordinates table (position.txt)
- raw and KNN-imputed RUD (APA usage) matrices
- morphological layer labels per spot (e.g., GCL, GL, ONL, OPL, MCL)

## Outputs

- spot cluster assignments mapped to tissue layer labels
- UMAP embedding of spots
- adjusted Rand index vs. layer labels (raw vs. imputed)
- median within-layer spot-spot Pearson correlation ranges of RUD values
- clustering_evaluation_metrics.csv

## How to apply

Load the movAPA PACdataset (APA.RDA), gene-spot count matrix (count.csv), and spot coordinates (position.txt) from the stAPAminer package extdata; compute raw APA usage with computeAPAIndex on the spots' barcodes, then impute with imputeAPAIndex using KNN (k=10, init=TRUE) borrowing from the spatial gene expression profile, setting any residual missing entries to 0. Build a stAPAminer object from the RUD matrix and position table with createStAPAminerObject, then run graph-based spot clustering with makeStCluster (e.g., resolution=0.6-0.7, dims=1:10, k=30, nfeatures=4500), assign layer labels with findLabels and RenameCluster (e.g., GCL, GL, ONL, OPL, MCL), and visualize with drawClusterPlot over a UMAP embedding. Evaluate agreement with morphological layer labels using computeIndex (adjusted Rand index) and within-layer spot-spot Pearson correlations of RUD values with computePearsonIndex(stObj,'RUD'); run the identical pipeline on the raw (unimputed) RUD matrix as a control to quantify imputation benefit.

## Related tools

- **stAPAminer** (provides createStAPAminerObject, makeStCluster, findLabels, RenameCluster, drawClusterPlot, computeIndex, computePearsonIndex for spot clustering and evaluation) — https://github.com/BMILAB/stAPAminer
- **Seurat** (underlying engine for graph-based clustering, dimensionality reduction, and UMAP embedding invoked by makeStCluster)
- **ggplot2** (visualization of cluster plots, UMAP, and volcano plots)
- **clusterSim** (supporting package for clustering evaluation indices)
- **ClusterR** (provides distinctColorPalette used for cluster plot coloring)

## Examples

```
stObj <- createStAPAminerObject(RUD,position); stObj <- makeStCluster(stObj,resolution = 0.7,dims = 1:10,k=30,nfeatures = 4500); index <- computeIndex(stObj)
```

## Evaluation signals

- Imputed-matrix spot clustering should reach ARI ≈ 0.643 vs. ≈ 0.547 for the raw RUD matrix against morphological layer labels.
- Median within-layer spot-spot Pearson correlations of imputed RUD values (0.525–0.622) should exceed raw values (0.311–0.501).
- Clustering parameters are checkable: resolution=0.6–0.7, dims=1:10, k=30, nfeatures=4500.
- The number of assigned cluster labels must match RenameCluster layer names (5 layers: GCL, GL, ONL, OPL, MCL) and spots must retain position.txt coordinates.
- Any APA ratio entries still missing after imputation iterations must be set to 0 before clustering.

## Limitations

- No method exists to determine whether an APA ratio matrix entry is truly missing before imputation; residual missing entries are arbitrarily set to 0.
- The APA ratio matrix is transformed into a count matrix to meet SPARK input requirements, which may not be theoretically reasonable or fully meet the model's assumptions.
- No version numbers are reported for the software tools, complicating exact reproduction.
- APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools, limiting applicability to image-based ST.

## Evidence

- [readme] graph-based spot clustering with makeStCluster parameters: [evidence span withheld — anchor status: external_doc]
- [methods] KNN imputation of APA index from spatial gene expression: "RUD <- imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [intro] imputation designed to recover APA signals by borrowing gene expression information: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [discussion] arbitrary zero-filling of residual missing entries: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [readme] correlation- and index-based verification of imputation: "pearson_RUD <- computePearsonIndex(stObj,"RUD")
index <- computeIndex(stObj)"
- [abstract] spatial APA usage across morphological layers of MOB: "By analyzing well-established ST data of the mouse olfactory bulb (MOB), we presented a detailed view of spatial APA usage across morphological layers of the MOB"
