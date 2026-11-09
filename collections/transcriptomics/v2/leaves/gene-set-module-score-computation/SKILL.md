---
name: gene-set-module-score-computation
description: Use when the goal is to convert annotated gene sets (pathways) into continuous
  per-spot or per-cell activity features for downstream KLD-based differential-activity
  testing, e.g., on Visium spatial transcriptomics data.
when_to_use_negative:
- 'Comparing multiple conditions (e.g., wild-type vs knockout): the downstream singleCellHaystack
  method cannot easily conduct comparisons between multiple conditions'
- 'When a gene set has fewer than 10 genes: the reference workflow filters msigdbr
  BioCarta sets to those with at least 10 genes before scoring'
- If the analysis requires cluster-oriented differential expression with condition
  labels rather than cluster-independent spatial activity detection
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides AddModuleScore to compute continuous per-spot pathway activity from
    gene sets over the Visium count matrix
- name: msigdbr
  role: Source of the 292 BioCarta gene set definitions used as scoring features
- name: singleCellHaystack
  role: Consumes the rescaled module-score feature matrix with 2D spot coordinates
    to predict pathways with non-random spatial distributions via KLD-based testing
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Language in which the scoring (Seurat, msigdbr) and testing (singleCellHaystack
    v1.0.0) workflow is implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e9f4a4fff7265d7313db0b92766b1bc1ce09baeea78ac72bb2edb74e44e76845
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - msigdbr
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-set-module-score-computation

## Summary

Computes continuous per-spot (or per-cell) pathway activity scores by applying Seurat AddModuleScore to curated gene sets (e.g., msigdbr BioCarta sets), producing a feature matrix that singleCellHaystack can use to find gene sets with non-random spatial distributions.

## When to use

Use when the goal is to convert annotated gene sets (pathways) into continuous per-spot or per-cell activity features for downstream KLD-based differential-activity testing, e.g., on Visium spatial transcriptomics data.

## When NOT to use

- Comparing multiple conditions (e.g., wild-type vs knockout): the downstream singleCellHaystack method cannot easily conduct comparisons between multiple conditions
- When a gene set has fewer than 10 genes: the reference workflow filters msigdbr BioCarta sets to those with at least 10 genes before scoring
- If the analysis requires cluster-oriented differential expression with condition labels rather than cluster-independent spatial activity detection

## Inputs

- Spot-level gene count matrix from a Visium spatial transcriptomics dataset (Seurat object)
- 2D spatial coordinates of Visium spots
- Gene set definitions from msigdbr (BioCarta collection), filtered to >= 10 genes
- Seurat R package for AddModuleScore

## Outputs

- Per-spot module score matrix (spots x gene sets) of continuous pathway activity values
- Z-normalized/rescaled module score feature matrix
- KLD-based p-values and FDR-adjusted p-values per pathway from singleCellHaystack
- Ranked list of differentially active pathways (e.g., top 25 per dataset) and pathway clusters

## How to apply

Load gene sets from msigdbr (292 BioCarta sets in the reference workflow) and filter to sets with at least 10 genes before scoring. For each gene set, run Seurat AddModuleScore over the spot-level count matrix of a Visium dataset (e.g., mouse anterior brain, posterior brain, kidney), yielding one continuous activity value per spot per pathway. Z-normalize (rescale) the resulting module scores so all pathway features are comparable, then use this rescaled score matrix as the feature matrix input to singleCellHaystack v1.0.0 together with the 2D spot coordinates as the input positions; singleCellHaystack uses Kullback-Leibler Divergence rather than cluster comparisons to detect non-random activity. Rank pathways by FDR-adjusted p-value (e.g., take the top 25 per dataset) and optionally cluster them with hclust_haystack for visualization.

## Related tools

- **Seurat** (Provides AddModuleScore to compute continuous per-spot pathway activity from gene sets over the Visium count matrix)
- **msigdbr** (Source of the 292 BioCarta gene set definitions used as scoring features)
- **singleCellHaystack** (Consumes the rescaled module-score feature matrix with 2D spot coordinates to predict pathways with non-random spatial distributions via KLD-based testing) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Language in which the scoring (Seurat, msigdbr) and testing (singleCellHaystack v1.0.0) workflow is implemented)

## Examples

```
seurat_obj <- AddModuleScore(object = seurat_obj, features = biocarta_gene_sets, name = "BioCarta_")
```

## Evaluation signals

- Every scored gene set retains >= 10 genes after the pre-scoring filter (292 BioCarta sets pass in the reference workflow)
- Module score matrix is continuous (not binary detection) and z-normalized before passing to singleCellHaystack
- Each pathway receives a KLD-based p-value and an FDR-adjusted p-value; top 25 pathways per dataset can be ranked
- Spatial plots of example high-scoring pathways and hclust_haystack pathway clusters reproduce patterns comparable to the published Fig. 5A-C
- Non-random (significant) pathways should show spatially coherent activity across Visium spots rather than spot-uniform distributions

## Limitations

- singleCellHaystack cannot easily conduct comparisons between multiple conditions (e.g., wild-type and knockout)
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- For spatial transcriptomics, singleCellHaystack has relatively longer runtimes than SPARK-X, though with better sensitivity

## Evidence

- [abstract] singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [intro] our approach can also be used on sets of genes for predicting differential activities of biological pathways: "our approach can also be used on sets of genes (e.g., genes sharing a common annotation) for predicting differential activities of biological pathways in single-cell and spatial transcriptomics data"
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space: [evidence span withheld — anchor status: external_doc]
- [discussion] One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted.: "One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted."
