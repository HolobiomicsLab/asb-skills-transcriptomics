---
name: spatial-clustering-of-st-spots
description: Use when an imputed APA index (RUD) matrix from spatially barcoded ST
  data (spots × poly(A) sites) needs to be partitioned into tissue layers, e.g., MOB
  sections, before layer-wise DEAPA or LSAPA tests.
when_to_use_negative:
- Input is raw poly(A)-site counts (APA.RDA) rather than a quantified and imputed
  APA-index matrix — quantification and KNN imputation must precede clustering
- Data are single-cell RNA-seq without spatial barcodes or coordinates — spots cannot
  be mapped to tissue layers and APA sites cannot be identified with scAPAtrap/Sierra
  from such data
- Image-based ST platforms (e.g., MERFISH/seqFISH) — the article notes APA sites can
  only be identified from spatially barcoded ST data with existing tools
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing createStAPAminerObject, makeStCluster, RenameCluster,
    findLabels, drawClusterPlot, and computePearsonIndex for clustering spots by APA
    usage and assigning MOB layers
  repo: https://github.com/BMILAB/stAPAminer
- name: Seurat
  role: Dependency underlying the Seurat-style clustering (dims, k, resolution, nfeatures
    parameters in makeStCluster)
- name: org.Mm.eg.db
  role: Required R package used to map ENTREZID to gene SYMBOL when building APA profiles
    per replicate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:ed4ddfb05df83c5c42d917437b587b3342507cb0a951be0808f13ec2075e9664
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - Seurat
  - movAPA
  - org.Mm.eg.db
  - scAPAtrap
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spatial-clustering-of-st-spots

## Summary

Cluster spatially barcoded ST spots by their imputed APA usage (RUD) profiles using stAPAminer's Seurat-based clustering (makeStCluster), then assign clusters to morphological layers (GCL, GL, ONL, OPL, MCL) so APA usage can be compared across MOB anatomy.

## When to use

Use when an imputed APA index (RUD) matrix from spatially barcoded ST data (spots × poly(A) sites) needs to be partitioned into tissue layers, e.g., MOB sections, before layer-wise DEAPA or LSAPA tests.

## When NOT to use

- Input is raw poly(A)-site counts (APA.RDA) rather than a quantified and imputed APA-index matrix — quantification and KNN imputation must precede clustering
- Data are single-cell RNA-seq without spatial barcodes or coordinates — spots cannot be mapped to tissue layers and APA sites cannot be identified with scAPAtrap/Sierra from such data
- Image-based ST platforms (e.g., MERFISH/seqFISH) — the article notes APA sites can only be identified from spatially barcoded ST data with existing tools

## Inputs

- Imputed APA usage (RUD) matrix: spots × poly(A) sites
- Spot coordinate table (position.txt: spot barcodes with x/y coordinates)
- Optional: spot-gene count matrix (count.csv) used for KNN imputation feeding the clustering

## Outputs

- Clustered stAPAminer object with spot-to-layer assignments (GCL, GL, ONL, OPL, MCL)
- Spatial cluster plot (drawClusterPlot) mapping clusters onto spot coordinates
- Within-layer vs between-layer Pearson correlation statistics validating imputation

## How to apply

First build the stAPAminer object from the imputed RUD matrix and the spot coordinate table with createStAPAminerObject(RUD, position). Cluster spots with makeStCluster using Seurat-style parameters — dims=1:10, k=30, nfeatures=4500, and a Louvain resolution of 0.6–0.7 (0.6 in the README example; 0.7 for Replicates 5 and 12 in the robustness task). Convert cluster IDs to MOB layer identities via RenameCluster (c("GCL","GL","ONL","OPL","MCL")), inspect with findLabels and drawClusterPlot, and validate that imputation improved layer separation by checking that within-layer spot correlations exceed between-layer correlations (computePearsonIndex with source='RUD'). Layer-informative clustering is the prerequisite for subsequent findContrastAPA/findFindAllMarkers calls, so clusters must map cleanly onto the known anatomical layers before proceeding.

## Related tools

- **stAPAminer** (R package providing createStAPAminerObject, makeStCluster, RenameCluster, findLabels, drawClusterPlot, and computePearsonIndex for clustering spots by APA usage and assigning MOB layers) — https://github.com/BMILAB/stAPAminer
- **Seurat** (Dependency underlying the Seurat-style clustering (dims, k, resolution, nfeatures parameters in makeStCluster))
- **org.Mm.eg.db** (Required R package used to map ENTREZID to gene SYMBOL when building APA profiles per replicate)

## Examples

```
stObj <- createStAPAminerObject(RUD,position); stObj <- makeStCluster(stObj,resolution = 0.6,dims = 1:10,k=30,nfeatures = 4500); findLabels(stObj)
```

## Evaluation signals

- Clusters map one-to-one onto the five known MOB morphological layers (GCL, GL, ONL, OPL, MCL) when renamed
- Imputed APA profiles show higher within-layer than between-layer Pearson correlations (computePearsonIndex source='RUD')
- Across MOB replicates, APA profiles correlate highly with Replicate 11 (r ≈ 0.87 for Replicate 5 and 0.86 for Replicate 12)
- Clustering parameters fall in the documented ranges: resolution 0.6–0.7, dims=1:10, k=30, nfeatures=4500
- Only spots with sufficient read support are included; the article retained spots with ten or more read counts (260 of 262 spots)

## Limitations

- Missing APA usage entries remaining after KNN imputation iterations are arbitrarily set to 0
- No method exists to determine whether an APA ratio matrix entry is truly missing before imputation
- Clustering quality depends on the imputation step, which borrows information from the spatial gene expression profile; no method is proposed to mitigate discrepancies among replicates
- README requires R (>=3.6.0, R 3.6.3 recommended); no version numbers are reported for any software tool used

## Evidence

- [readme] stObj <- createStAPAminerObject(RUD,position)
stObj <- makeStCluster(stObj,resolution = 0.6,dims = 1:10,k=30,nfeatures = 4500)
findLabels(stObj): [evidence span withheld — anchor status: external_doc]
- [readme] The input of stAPAminer is a poly(A) site matrix with each row being a poly(A) site and each column being a spot.: [evidence span withheld — anchor status: external_doc]
- [readme] By analyzing the well-established ST data of mouse olfactory bulb (MOB), we present a detailed view of spatial APA usage across morphological layers of MOB with stAPAminer.: [evidence span withheld — anchor status: external_doc]
- [intro] an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data"
- [discussion] We retained spots with ten or more read counts, resulting in 260 spots: "We retained spots with ten or more read counts, resulting in 260 spots"
