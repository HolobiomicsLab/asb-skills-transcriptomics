---
name: anndata-object-handling
description: Use when integrating unpaired scRNA-seq and scATAC-seq data with scBridge
  and you need to prepare, subset, or extract results from h5ad AnnData objects (cell
  types in obs, embeddings in obsm).
when_to_use_negative:
- Inputs are raw 10x fragment files or peak-by-cell matrices not yet converted to
  a gene activity matrix — transform with Signac first before building the target
  AnnData
- Data are paired multi-omics (e.g. SNARE-seq, 10x Multiome) where joint integration
  methods like scAI or MOFA+ that exploit cross-omics pairing are more appropriate
- Analysis is done entirely in R/Bioconductor (e.g. Signac, Seurat) without converting
  to h5ad — scBridge only accepts AnnData inputs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scBridge
  role: Consumes source/target AnnData h5ad files and returns Prediction, Reliability,
    and Embedding fields on the integrated target AnnData
  repo: https://github.com/XLearning-SCU/scBridge
- name: Signac
  role: Transforms the scATAC-seq peak-by-cell matrix into a gene activity matrix
    before AnnData assembly
  repo: https://github.com/stuart-lab/signac
- name: harmonypy
  role: Harmonizes the obsm['Embedding'] joint embedding extracted from the target
    AnnData prior to silhouette evaluation
  repo: https://github.com/slowkow/harmonypy
- name: scikit-learn
  role: Provides metrics (F1-score, silhouette) and the Gaussian Mixture Model applied
    to reliability scores stored on the AnnData
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:e9cec949eeb05395fc4facbe2148f34c245357577da21d8d8ad87ee7887a2550
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch
  - Python
  - Signac
  - harmonypy
  - scBridge
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# anndata-object-handling

## Summary

Structure scBridge's single-cell multi-omics inputs and outputs as AnnData objects: source scRNA-seq and target scATAC-seq h5ad files carry cell types, peak-derived gene activity, common-gene subsetting, and results (Prediction, Reliability, Embedding) in .obs/.obsm slots.

## When to use

Use when integrating unpaired scRNA-seq and scATAC-seq data with scBridge and you need to prepare, subset, or extract results from h5ad AnnData objects (cell types in obs, embeddings in obsm).

## When NOT to use

- Inputs are raw 10x fragment files or peak-by-cell matrices not yet converted to a gene activity matrix — transform with Signac first before building the target AnnData
- Data are paired multi-omics (e.g. SNARE-seq, 10x Multiome) where joint integration methods like scAI or MOFA+ that exploit cross-omics pairing are more appropriate
- Analysis is done entirely in R/Bioconductor (e.g. Signac, Seurat) without converting to h5ad — scBridge only accepts AnnData inputs

## Inputs

- Source scRNA-seq AnnData h5ad with cell-type annotations in obs['CellType'] (e.g. Tabula Muris FACS/droplet count matrices)
- Target scATAC-seq AnnData h5ad with gene activity or peak-by-cell matrix (e.g. GSE111586 mouse atlas)
- Common-gene list for subsetting both modalities (e.g. 12,689 common genes, 18 common cell types)

## Outputs

- Target AnnData with transferred labels in obs['Prediction']
- Target AnnData with per-cell confidence scores in obs['Reliability']
- Joint cross-omics embedding in obsm['Embedding']
- Source/target AnnData normalized via Standard (scRNA-seq) and TF-IDF scaled to 10,000 counts per cell (scATAC-seq)

## How to apply

Represent the annotated scRNA-seq source as an AnnData h5ad with cell-type annotations in obs['CellType'], and the scATAC-seq target as an AnnData whose peak-by-cell matrix has first been transformed into a gene activity matrix (e.g. with Signac), keeping only the common genes. Restrict both objects to shared cell types and the 12,689 common genes for atlas-scale runs such as Tabula Muris plus the mouse atlas scATAC-seq (GSE111586, ~102,103 cells). After running scBridge, read transferred labels and per-cell confidence from obs['Prediction'] and obs['Reliability'] on the target AnnData, and pull the joint cross-omics embedding from obsm['Embedding'] for downstream silhouette-score evaluation or harmonypy harmonization. Keep normalization metadata attached (Standard for scRNA-seq, TF-IDF with 10,000-count scaling for scATAC-seq) so the preprocessing state travels with the object.

## Related tools

- **scBridge** (Consumes source/target AnnData h5ad files and returns Prediction, Reliability, and Embedding fields on the integrated target AnnData) — https://github.com/XLearning-SCU/scBridge
- **Signac** (Transforms the scATAC-seq peak-by-cell matrix into a gene activity matrix before AnnData assembly) — https://github.com/stuart-lab/signac
- **harmonypy** (Harmonizes the obsm['Embedding'] joint embedding extracted from the target AnnData prior to silhouette evaluation) — https://github.com/slowkow/harmonypy
- **scikit-learn** (Provides metrics (F1-score, silhouette) and the Gaussian Mixture Model applied to reliability scores stored on the AnnData)

## Examples

```
import anndata as ad; src = ad.read_h5ad('tabula_muris_rna.h5ad'); tgt = ad.read_h5ad('mouse_atac_activity.h5ad'); labels = tgt.obs['Prediction']; emb = tgt.obsm['Embedding']
```

## Evaluation signals

- obs['CellType'] is present and non-null on the source h5ad before running scBridge
- Both source and target AnnData are restricted to the common cell types and 12,689 common genes for the mouse atlas comparison
- After integration, obs['Prediction'] and obs['Reliability'] exist on the target AnnData and Reliability values fall in [0,1]
- obsm['Embedding'] has one row per target cell and supports silhouette-score harmonization matching Fig. 3 values (scBridge ~89.66% F1 vs scJoint 64.60%, Seurat 44.64%)

## Limitations

- scBridge accepts only AnnData (.h5ad) inputs; scATAC-seq peak data must be pre-converted to a gene activity matrix with tools such as Signac before an AnnData target can be built
- Peak-by-cell matrices must be sourced separately (e.g. GSE111586 from Zenodo records 6578553/6578617) since atlas sites provide only unnormalized gene activity matrices
- Only cells of common cell types and common genes between modalities can be retained; cells annotated 'Unknown' must be excluded or they break label transfer

## Evidence

- [readme] scBridge accepts AnnData as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files.: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
- [readme] Source AnnData must carry cell types in obs['CellType']; peaks need conversion to an activity matrix.: "The source data should include the cell type annotation in `obs.["CellType"]`. For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [full_text] Results are extracted from .obs['Prediction'] and .obs['Reliability'] with joint embeddings in .obsm['Embedding'].: [evidence span withheld — anchor status: not_found]
- [discussion] Atlas-scale AnnData objects were subset to 18 common cell types and 12,689 common genes.: "We reserved cells of 18 common types and 12,689 common genes for the analysis"
- [methods] Normalization strategy (Standard/TF-IDF, 10,000 scaling) must be applied per modality before integration.: "`--source_preprocess=["Standard", "TFIDF"]` (Default="Standard") the preprocessing strategy for source data"
