---
name: spatial-kde-density-weighting
description: Use when analyzing spatial transcriptomics data (e.g., 10x Visium stxBrain
  anterior1) to find GO/pathway gene sets whose activity is spatially organized across
  or within transcriptomic clusters, including high-order patterns missed by cluster-centric
  analysis.
when_to_use_negative:
- Input is targeted gene data with limited numbers of genes (e.g., targeted panels)
  — GSDensity spatial analysis is only applicable to spatial whole transcriptomic
  data; imputation of unmeasured genes is suggested but not implemented
- The dataset has no spatial coordinate information (plain scRNA-seq) — use the density/coordination
  (compute.kld) and label propagation workflow instead
- Gene sets have not passed the coordination test — binarization/propagation downstream
  steps are recommended only on gene sets passing the KLD cutoff
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: Provides compute.spatial.kld.df, compute.spatial.kld, and compute.spec for
    weighted-KDE spatial relevance testing and specificity scoring
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: Loads and normalizes the Visium spatial dataset and supplies spatial coordinates
    and spot annotations
- name: SeuratData
  role: Source of the stxBrain anterior1 Visium mouse forebrain dataset
- name: msigdbr
  role: Supplies the GO Biological Process gene set collection tested for spatial
    relevance
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:0ca96f84db4c5c56a02ad10ec3f62e834e015bd4520012f2ca1ffa80f00a31be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - SeuratData
  - msigdbr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-kde-density-weighting

## Summary

GSDensity's weighted-KDE approach for identifying spatially relevant pathways: pathway activity scores from random-walk propagation are used as cell weights in a 2D KDE on spatial coordinates, and the weighted density is tested against an equal-weighted null via permutation-based KL divergence (compute.spatial.kld / compute.spatial.kld.df).

## When to use

Use when analyzing spatial transcriptomics data (e.g., 10x Visium stxBrain anterior1) to find GO/pathway gene sets whose activity is spatially organized across or within transcriptomic clusters, including high-order patterns missed by cluster-centric analysis.

## When NOT to use

- Input is targeted gene data with limited numbers of genes (e.g., targeted panels) — GSDensity spatial analysis is only applicable to spatial whole transcriptomic data; imputation of unmeasured genes is suggested but not implemented
- The dataset has no spatial coordinate information (plain scRNA-seq) — use the density/coordination (compute.kld) and label propagation workflow instead
- Gene sets have not passed the coordination test — binarization/propagation downstream steps are recommended only on gene sets passing the KLD cutoff

## Inputs

- Seurat spatial object (e.g., stxBrain anterior1 Visium dataset) with normalized expression matrix and 2D spatial coordinates
- MCA co-embedding of cells and genes from compute.mca
- List of coordinated gene sets (e.g., GO Biological Process from msigdbr) passing compute.kld significance (adjusted p < 0.05)
- Per-cell pathway activity weights from run.rwr.list (random walk with restart on the cell-gene graph)
- Spot/cell cluster annotations or labels for compute.spec

## Outputs

- Spatial relevance statistics per gene set: permutation-based KLD p-values and adjusted p-values (< 0.05 cutoff)
- Cluster-wise specificity scores per gene set (Jensen-Shannon divergence based, compute.spec)
- Joint scatter plot of spatial relevance vs cluster-wise specificity highlighting high-specificity and high-relevance/low-specificity terms (e.g., visium_anterior1_joint_plot.png)

## How to apply

After compute.mca co-embedding and compute.kld coordination screening (adjusted p < 0.05), build the nearest-neighbor cell-gene graph with compute.nn.edges (e.g., nn.use = 300) and score pathway activity per cell with run.rwr.list using restart = 0.75. Then test spatial relevance with compute.spatial.kld.df, which compares the 2D KDE of spatial coordinates weighted by pathway activity against an equal-weighted KDE, assessing significance with a permutation-based null over n.times replicates. Independently quantify cluster-wise specificity with compute.spec, a Jensen-Shannon divergence based score using the spot annotations. Plot spatial relevance vs specificity to highlight high-specificity terms and high-relevance/low-specificity terms (e.g., 'positive regulation of cellular response to insulin stimulus' near the caudal side and 'fatty acid oxidation' near the ventral side of the mouse anterior brain), revealing spatial organization spanning multiple clusters.

## Related tools

- **GSDensity** (Provides compute.spatial.kld.df, compute.spatial.kld, and compute.spec for weighted-KDE spatial relevance testing and specificity scoring) — https://github.com/qingnanl/gsdensity
- **Seurat** (Loads and normalizes the Visium spatial dataset and supplies spatial coordinates and spot annotations)
- **SeuratData** (Source of the stxBrain anterior1 Visium mouse forebrain dataset)
- **msigdbr** (Supplies the GO Biological Process gene set collection tested for spatial relevance)

## Examples

```
el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr.list(el, gene_set_list, cells, restart = 0.75); spatial.kld <- compute.spatial.kld.df(coembed, cv, gene.set.deviated, n.times = 100); spec <- compute.spec(cell_df, metadata, cell_group)
```

## Evaluation signals

- Adjusted permutation-based KLD p-values for spatially relevant gene sets must be < 0.05 (compute.spatial.kld.df output)
- Joint plot must display coordinated GO BP terms with highlighted high-specificity and high-relevance/low-specificity categories, matching the structure of the published spatial vignette figure (visium_anterior1_joint_plot.png)
- Weighted KDE must be computed with pathway-activity weights (run.rwr outputs) while the null uses equal weights — verify weights are correctly aligned to spatial spots
- Expected biological anchors: 'positive regulation of cellular response to insulin stimulus' enriched near the caudal side and 'fatty acid oxidation' near the ventral side of the mouse anterior brain
- High-relevance/low-specificity terms should exhibit high spatial relevance but low compute.spec (Jensen-Shannon divergence) specificity, indicating cross-cluster spatial organization

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset)
- gsdensity version number not stated; CRAN install temporarily unavailable, requiring GitHub/remotes installation and an older anticlust version (e.g., 0.6.1)

## Evidence

- [intro] the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [methods] Now we provide statistical test for spatial relevance of gene sets: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [discussion] Weighted-KDE based identification of spatially relevant pathways: "we also expanded GSDensity to identify spatially relevant pathways"
- [readme] The latter one is mainly focused on how to associate gene sets with the spatial information: "The latter one is mainly focused on how to associate gene sets with the spatial information of cells or other partition of cells."
