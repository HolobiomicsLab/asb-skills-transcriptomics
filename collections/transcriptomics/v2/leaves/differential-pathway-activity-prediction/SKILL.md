---
name: differential-pathway-activity-prediction
description: Use when you have spot-level (or cell-level) continuous feature values
  such as Seurat module scores plus 2D/3D spatial coordinates, and you need to rank
  gene sets or features by how non-randomly their activity is distributed across the
  tissue.
when_to_use_negative:
- Comparisons between multiple conditions (e.g., wild-type and knockout) — singleCellHaystack
  cannot easily conduct multi-condition comparisons, per the authors' discussion
- When the Python implementation is required and hierarchical clustering of gene sets
  is needed — the Python package lacks the hclust_haystack cluster_genes method
- When minimal runtime on large spatial data is the priority — SPARK-X is faster,
  though singleCellHaystack is more sensitive
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: KLD-based prediction of differentially active pathways from rescaled module
    scores and 2D Visium spot coordinates (R package v1.0.0+)
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: AddModuleScore computes continuous per-spot pathway activity values for each
    BioCarta gene set
- name: msigdbr
  role: Source of the 292 BioCarta gene sets, filtered to sets with at least 10 genes
    before scoring
- name: singleCellHaystack-py
  role: Python implementation (v0.0.5, beta) of the same KLD method; missing cluster_genes/hclust_haystack
    used in this workflow
  repo: https://github.com/ddiez/singleCellHaystack-py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:33810752b79d3142a9021e507753b5fdf8cdea412b903427845e624d1aa07225
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - msigdbr
  - singleCellHaystack
  - singleCellHaystack-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# differential-pathway-activity-prediction

## Summary

Predict biological pathways (gene sets) with non-random spatial distributions in Visium spatial transcriptomics data by scoring msigdbr BioCarta gene sets with Seurat AddModuleScore and running singleCellHaystack v1.0.0 KLD-based detection on spot coordinates, without cluster-based comparisons.

## When to use

Use when you have spot-level (or cell-level) continuous feature values such as Seurat module scores plus 2D/3D spatial coordinates, and you need to rank gene sets or features by how non-randomly their activity is distributed across the tissue.

## When NOT to use

- Comparisons between multiple conditions (e.g., wild-type and knockout) — singleCellHaystack cannot easily conduct multi-condition comparisons, per the authors' discussion
- When the Python implementation is required and hierarchical clustering of gene sets is needed — the Python package lacks the hclust_haystack cluster_genes method
- When minimal runtime on large spatial data is the priority — SPARK-X is faster, though singleCellHaystack is more sensitive

## Inputs

- Visium spatial transcriptomics spot-level count matrices (mouse anterior brain, posterior brain, kidney tutorial datasets)
- 2D spot spatial coordinates (x, y positions per spot)
- msigdbr BioCarta gene sets (292 sets, filtered to ≥10 genes)
- Seurat object for AddModuleScore-based pathway scoring

## Outputs

- KLD-based p-values and FDR-adjusted p-values per pathway (singleCellHaystack result object)
- Ranked list of top 25 differentially active pathways per dataset
- Pathway clusters from hclust_haystack hierarchical clustering
- Spatial plots of pathway activity distributions across Visium spots (comparable to Fig 5A-C)

## How to apply

Load the mouse Visium datasets (anterior brain, posterior brain, kidney) with spot-level counts and 2D spatial coordinates. Pull the 292 msigdbr BioCarta gene sets, filter to those with at least 10 genes, and score each set in every spot with Seurat AddModuleScore to get continuous per-spot pathway activity values. Z-normalize (rescale) the module scores, then run singleCellHaystack v1.0.0 with the 2D spot coordinates as input positions and the rescaled module scores as the feature matrix; singleCellHaystack uses Kullback-Leibler Divergence to find features whose activity patterns occupy non-randomly positioned subsets of the input space. Rank pathways by FDR-adjusted p-value per dataset, take the top 25 differentially active pathways, cluster them with the hclust_haystack hierarchical clustering method, and plot the spatial distribution of example high-scoring pathways across tissue spots. The rationale for using module scores as continuous input is that singleCellHaystack v1.0+ accepts continuous features (unlike the original binary-threshold version), avoiding overestimation of significance from a hard detection cutoff.

## Related tools

- **singleCellHaystack** (KLD-based prediction of differentially active pathways from rescaled module scores and 2D Visium spot coordinates (R package v1.0.0+)) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (AddModuleScore computes continuous per-spot pathway activity values for each BioCarta gene set)
- **msigdbr** (Source of the 292 BioCarta gene sets, filtered to sets with at least 10 genes before scoring)
- **singleCellHaystack-py** (Python implementation (v0.0.5, beta) of the same KLD method; missing cluster_genes/hclust_haystack used in this workflow) — https://github.com/ddiez/singleCellHaystack-py

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Each retained gene set has ≥10 genes after the msigdbr BioCarta filter before AddModuleScore
- Module scores are z-normalized (rescaled) before being passed to haystack; positions argument contains the 2D spot coordinates
- Output contains KLD-based p-values per pathway; after Benjamini-Hochberg FDR correction, top 25 pathways per dataset should match the paper's ranking
- hclust_haystack clustering of the top 25 pathways and spatial plots of example high-scoring pathways should visually reproduce published Fig 5A-C for the anterior brain, posterior brain, and kidney Visium datasets
- Pathways flagged as significant should show spatially coherent (non-random) activity patterns across tissue spots when plotted

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses, compared to cluster-based DEG prediction approaches
- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, though it offers better sensitivity
- The Python implementation (v0.0.5, beta) is missing weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes, so the R package is required for the hclust_haystack step

## Evidence

- [abstract] singleCellHaystack now accepts continuous features (RNA or protein expression, chromatin accessibility, module scores) from single-cell, spatial, and bulk genomics data: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [other] Workflow: score BioCarta gene sets and run haystack on coordinates: [evidence span withheld — anchor status: not_found]
- [other] Workflow: haystack inputs are z-normalized module scores and 2D coordinates: [evidence span withheld — anchor status: not_found]
- [readme] KLD mechanism, no clustering required: [evidence span withheld — anchor status: external_doc]
- [readme] Python implementation lacks the clustering method used in step 5: "Hierarchical clustering method for `cluster_genes`."
