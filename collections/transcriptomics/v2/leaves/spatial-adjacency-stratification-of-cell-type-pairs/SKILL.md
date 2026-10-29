---
name: spatial-adjacency-stratification-of-cell-type-pairs
description: Use when you have spatial transcriptomics data with known cell-type labels
  (e.g., seqFISH mouse organogenesis, 10x Visium mouse brain) plus a source–target
  CCI probability matrix, and you need to test whether predicted interaction strengths
  track physical spatial proximity.
when_to_use_negative:
- Input is non-spatial scRNA-seq data (e.g., Yan GSE36552 or Quake_10x_Limb_Muscle
  GSE109774) with no spatial coordinates — the adjacent/distant stratification cannot
  be computed
- Cell types are unlabeled and the CCI predictions are for unlabeled clusters rather
  than labeled source/target cell types
- You only need interaction inference itself, not validation of predicted probabilities
  against spatial proximity
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: GCN-based framework whose Interaction model produces the source-to-target
    cell-type interaction probability outfile being validated against spatial adjacency
  repo: https://github.com/JiangBioLab/DeepCCI
- name: Seurat
  role: R package used for visualization of adjacent-vs-distant interaction score
    figures in the Fig. 4c-f style
- name: igraph
  role: R package used alongside Seurat for rendering CCI network/score visualizations
    of adjacent and distant cell-type pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:cb47ad968309fa2b872625f5ba26c92b92f13f3df2869c26d13fa6849e28ab04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - DeepCCI
  - Python
  - R
  - Seurat
  - igraph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Spatial adjacency stratification of cell-type pairs

## Summary

Validates spatially resolved cell-cell interaction (CCI) predictions by using spatial coordinates to compute average pairwise distances between cell types, splitting source–target pairs into spatially adjacent versus distant strata, and checking that predicted interaction probabilities are systematically stronger for adjacent pairs (e.g., Card adjacent to FMH in seqFISH) than for distant ones (e.g., SC distant from FMH).

## When to use

Use when you have spatial transcriptomics data with known cell-type labels (e.g., seqFISH mouse organogenesis, 10x Visium mouse brain) plus a source–target CCI probability matrix, and you need to test whether predicted interaction strengths track physical spatial proximity.

## When NOT to use

- Input is non-spatial scRNA-seq data (e.g., Yan GSE36552 or Quake_10x_Limb_Muscle GSE109774) with no spatial coordinates — the adjacent/distant stratification cannot be computed
- Cell types are unlabeled and the CCI predictions are for unlabeled clusters rather than labeled source/target cell types
- You only need interaction inference itself, not validation of predicted probabilities against spatial proximity

## Inputs

- Spatially resolved expression data with cell-type labels (seqFISH mouse organogenesis FMH source; 10x Visium mouse brain C4 source) from the DeepCCI example test files
- Spatial coordinates per cell/spot
- DeepCCI-predicted source-to-target cell-type interaction probability outfile (from Interaction_inference.py)

## Outputs

- Average pairwise spatial distance per cell-type pair, stratified into adjacent vs distant groups
- Correlation of recomputed interaction probabilities with average spatial distances across cell-type pairs
- Adjacent-vs-distant interaction score figures in the style of Fig. 4c-f and Supplementary Figs. S6-S7 (Seurat/igraph)

## How to apply

Load the labeled spatial expression data (FMH from seqFISH mouse organogenesis; C4 from 10x Visium mouse brain) from the DeepCCI example test files (http://jianglab.org.cn/deepcci_download/), then run Feature.py with --label_mode True --species Human to prepare labeled inputs for the interaction model, followed by Interaction_inference.py --device cuda to obtain the source-to-target cell-type interaction probability outfile. Using the spatial coordinates, compute average pairwise distances between each source and target cell type and bin cell-type pairs into spatially adjacent versus distant groups for each dataset. Correlate the recomputed interaction probabilities with the average spatial distances across cell-type pairs, comparing adjacent versus distant strata within the FMH and C4 datasets. Visualize adjacent-vs-distant interaction scores in the style of Fig. 4c-d and 4e-f (plus Supplementary Figs. S6-S7) with the R stack (Seurat/igraph), accepting the recomputation only if adjacent pairs receive systematically higher interaction scores than distant pairs in both datasets.

## Related tools

- **DeepCCI** (GCN-based framework whose Interaction model produces the source-to-target cell-type interaction probability outfile being validated against spatial adjacency) — https://github.com/JiangBioLab/DeepCCI
- **Seurat** (R package used for visualization of adjacent-vs-distant interaction score figures in the Fig. 4c-f style)
- **igraph** (R package used alongside Seurat for rendering CCI network/score visualizations of adjacent and distant cell-type pairs)

## Examples

```
cd Interaction_model; python Feature.py --label_mode True --species Human; python Interaction_inference.py --device cuda
```

## Evaluation signals

- On seqFISH mouse organogenesis, DeepCCI consistently assigns stronger interaction probabilities to spatially adjacent pairs (e.g., Card adjacent to FMH) than to distant pairs (e.g., SC distant from FMH)
- The same adjacent > distant pattern holds in the 10x Visium mouse brain dataset (C4 source) — both FMH and C4 datasets must show the pattern for acceptance
- Recomputed results match the paper's Fig. 4c-f and Supplementary Figs. S6-S7
- Interaction probabilities are negatively associated with average pairwise spatial distances across cell-type pairs within each dataset
- The interaction inference outfile is regenerated from the DeepCCI example test files with --device cuda (runs in about 1-2 minutes per the README)

## Limitations

- Validation depends on external labeled spatial datasets (FMH, C4) downloaded from the DeepCCI example site; it cannot be run on unlabeled or non-spatial data
- The paper's README/pipeline does not document a dedicated spatial-validation script — distance computation and stratification must be implemented externally from the spatial coordinates
- DeepCCI was demonstrated on scRNA-seq datasets from diverse technologies, but the article's discussion section text was not available, so scope caveats beyond the spatial benchmark are undocumented
- Interaction inference is GPU-dependent (--device cuda) in the documented workflow

## Evidence

- [intro] DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data.: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data."
- [readme] The example test file can be download from  http://jianglab.org.cn/deepcci_download/.: "The example test file can be download from  http://jianglab.org.cn/deepcci_download/."
- [readme] The predicted interaction outfile will be provided. The predicted process will take about 1-2 minutes.: "The predicted interaction outfile will be provided. The predicted process will take about 1-2 minutes."
- [methods] python Feature.py --label_mode True --species Human: "python Feature.py --label_mode True --species Human"
