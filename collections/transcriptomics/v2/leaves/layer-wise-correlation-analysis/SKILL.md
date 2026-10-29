---
name: layer-wise-correlation-analysis
description: Use when you have a Seurat-clustered stAPAminer object (MOB spots labeled
  into layers like GCL, GL, ONL, OPL, MCL) and need a per-layer correlation score
  to verify imputation quality or benchmark two imputed RUD matrices.
when_to_use_negative:
- The APA usage matrix has not yet been spatially clustered into morphological layers,
  since layer-wise correlation requires within-layer spot groupings
- You want to evaluate raw (unimputed) spatial gene expression rather than imputed
  APA usage ratios — use spatially variable gene tools like SPARK or SpatialDE instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: stAPAminer
  role: Provides computePearsonIndex, createStAPAminerObject, makeStCluster, findLabels
    and RenameCluster used to cluster spots into MOB layers and compute within-layer
    RUD correlations
  repo: https://github.com/BMILAB/stAPAminer
- name: Seurat
  role: Underlies the makeStCluster clustering of spots whose layer labels define
    the correlation groups
- name: ggplot2
  role: Draws the grouped boxplot (fill=source) comparing layer-wise correlation metrics
    between imputation methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:3289e738a03437e3b4b8bb7920b0d675be146c07bdee53c87daa568c15f9ef22
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
  - scDaPars
  - stAPAminer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# layer-wise-correlation-analysis

## Summary

Quantifies within-layer (within-cluster) Pearson correlations of spot-level APA usage (RUD) values from a stAPAminer object to score how well imputed spatial APA signals co-vary among spots in the same MOB morphological layer, serving as a quality metric for comparing imputation methods such as stAPAminer KNN vs scDaPars.

## When to use

Use when you have a Seurat-clustered stAPAminer object (MOB spots labeled into layers like GCL, GL, ONL, OPL, MCL) and need a per-layer correlation score to verify imputation quality or benchmark two imputed RUD matrices.

## When NOT to use

- The APA usage matrix has not yet been spatially clustered into morphological layers, since layer-wise correlation requires within-layer spot groupings
- You want to evaluate raw (unimputed) spatial gene expression rather than imputed APA usage ratios — use spatially variable gene tools like SPARK or SpatialDE instead

## Inputs

- stAPAminer object (stObj) holding an imputed RUD APA-usage matrix, spot coordinates from position.txt, and cluster labels from makeStCluster/findLabels
- Imputed APA usage (RUD) spot-by-poly(A)-site ratio matrix (source='RUD')
- Cluster-to-layer label mapping, e.g. c("GCL","GL","ONL","OPL","MCL")

## Outputs

- Per-layer within-cluster Pearson correlation values for the imputed RUD matrix (pearson_RUD)
- Comparison-ready correlation metrics per imputation method for ggplot2 grouped boxplots (fill=source) and aggregated metrics table

## How to apply

First cluster the imputed APA usage (RUD) matrix by building a stAPAminer object from the imputed matrix and spot coordinates, running makeStCluster (e.g. resolution=0.7, dims=1:10, k=30, nfeatures=4500) and findLabels, then renaming clusters to MOB layers such as GCL, GL, ONL, OPL, MCL. Then call computePearsonIndex with source='RUD' to compute, for each morphological layer, Pearson correlations of APA usage among spots within that layer; high within-layer correlation indicates the imputation recovered coherent layer-specific APA signals. Repeat identically on each imputation variant (stAPAminer KNN k=10 imputeAPAIndex vs scDaPars nearest-neighbor filling) and aggregate the correlation values with computeIndex results into a metrics table for grouped boxplot comparison. The rationale is that true spatial APA structure should make spots in the same MOB layer more similar to each other than to spots in other layers, so layer-wise correlation is a direct, label-based check of imputation correctness.

## Related tools

- **stAPAminer** (Provides computePearsonIndex, createStAPAminerObject, makeStCluster, findLabels and RenameCluster used to cluster spots into MOB layers and compute within-layer RUD correlations) — https://github.com/BMILAB/stAPAminer
- **Seurat** (Underlies the makeStCluster clustering of spots whose layer labels define the correlation groups)
- **ggplot2** (Draws the grouped boxplot (fill=source) comparing layer-wise correlation metrics between imputation methods)

## Examples

```
stObj <- createStAPAminerObject(RUD,position); stObj <- makeStCluster(stObj,resolution=0.7,dims=1:10,k=30,nfeatures=4500); findLabels(stObj); pearson_RUD <- computePearsonIndex(stObj,"RUD")
```

## Evaluation signals

- The stAPAminer-imputed RUD matrix should yield higher within-layer Pearson correlations than the scDaPars-imputed matrix, consistent with Figure 3E/F showing stAPAminer outperforming scDaPars regardless of performance indicator
- Correlation computation must use the imputed RUD matrix (source='RUD'), not raw gene counts, and must be restricted to spots sharing the same layer label (e.g. all GCL spots together)
- Cluster labels must match the five expected MOB morphological layers (GCL, GL, ONL, OPL, MCL) after RenameCluster before correlations are computed
- The same clustering parameters (resolution=0.7, dims=1:10, k=30, nfeatures=4500) must be applied to both imputation variants so correlation scores are comparable

## Limitations

- The layer-wise correlation is only as good as the clustering: misspecified resolution or dims in makeStCluster can split or merge layers and distort the correlation groups
- APA usage entries still missing after KNN imputation iterations are set directly to 0, which can bias within-layer correlations
- It is crucial to determine whether an entry into the APA ratio matrix is truly missing before imputation; no such method is provided, so spurious correlations may be introduced

## Evidence

- [readme] Verify by correlation
pearson_RUD <- computePearsonIndex(stObj,"RUD"): "##Verify by correlation
pearson_RUD <- computePearsonIndex(stObj,"RUD")"
- [results] stAPAminer outperformed scDaPars: [evidence span withheld — anchor status: not_found]
- [abstract] an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals: "APA sites were identified and quantified from the ST data. In particular, an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals"
- [readme] cluster renaming to MOB layers: "stObj <- RenameCluster(stObj,c("GCL","GL","ONL","OPL","MCL"))"
