---
name: co-expression-attractor-feature-selection
description: Use when clustering scRNA-seq data (normalized, log-transformed expression
  matrix of genes x cells) and you want biologically meaningful co-expression-derived
  features (e.g. attractor gene sets seeded by markers like AIF1) to feed clustering
  rather than raw or PCA-only features.
when_to_use_negative:
- Input data is raw counts or otherwise not normalized/log-transformed — CASCC.featureSelection
  expects 'well-normalized' input
- Input is already a reduced feature/embedding table (e.g. PCA or CCA components)
  rather than a gene x cell expression matrix
- You need a quick numeric-only clustering with no biological feature interpretation
  — generic k-means on PCA features suffices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASCC
  role: R package providing CASCC.featureSelection and findAttractor.adaptive for
    co-expression attractor feature selection
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency implementing the adaptive attractor algorithm underlying co-expression
    feature identification
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat 4.3.0
  role: Pinned dependency; provides the scRNA-seq data structure and downstream DEG
    analysis (FindAllMarkers) on attractor-defined clusters
  repo: https://github.com/LingyiC/CASCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:30979212247862f5cfb557f130cb892a504313652efb0b8d479987523421cbf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - NbClust
  - R
  - Seurat 4.3.0
  - aricode
  - cafr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# co-expression-attractor-feature-selection

## Summary

Selects gene co-expression features for single-cell RNA-seq clustering using the CASCC adaptive attractor algorithm, replacing generic feature spaces with biologically interpretable co-expression modules before clustering.

## When to use

Use when clustering scRNA-seq data (normalized, log-transformed expression matrix of genes x cells) and you want biologically meaningful co-expression-derived features (e.g. attractor gene sets seeded by markers like AIF1) to feed clustering rather than raw or PCA-only features.

## When NOT to use

- Input data is raw counts or otherwise not normalized/log-transformed — CASCC.featureSelection expects 'well-normalized' input
- Input is already a reduced feature/embedding table (e.g. PCA or CCA components) rather than a gene x cell expression matrix
- You need a quick numeric-only clustering with no biological feature interpretation — generic k-means on PCA features suffices

## Inputs

- Normalized, log-transformed scRNA-seq expression matrix (genes x cells), e.g. a Tabula Sapiens organ dataset or the bundled Data_PDAC_peng_2k
- inputDataType flag, 'well-normalized' for already preprocessed data
- Optional seed marker gene name for findAttractor.adaptive (e.g. 'AIF1')
- Optional prior attractor result (attr.raw = NULL by default)

## Outputs

- fs.res feature-selection result object consumed by predictClusterK and kmeansCenterModify
- Attractor co-expression modules (attractor list with top co-expression features per attractor)
- Final cluster assignments res$mainType.output$clusteringResults after downstream clustering

## How to apply

Install the CASCC R package with its pinned dependencies (Seurat 4.3.0 and cafr from GitHub). Preprocess the scRNA-seq matrix as normalized, log-transformed expression so it can be passed with inputDataType 'well-normalized'; CASCC.featureSelection exposes parameters such as exponent.max = 10 and exponent.min = 2 bounding the adaptive attractor search. Optionally seed the attractor search with a known marker via findAttractor.adaptive (e.g. 'AIF1'); helper functions filterGenes, scanSeeds.adaptive, collapseAttractorList, and attractorModify refine the attractor list. The returned fs.res object is then consumed by predictClusterK (ward.D2 linkage, silhouette index) and kmeansCenterModify to produce final cluster assignments in res$mainType.output$clusteringResults.

## Related tools

- **CASCC** (R package providing CASCC.featureSelection and findAttractor.adaptive for co-expression attractor feature selection) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency implementing the adaptive attractor algorithm underlying co-expression feature identification) — https://github.com/weiyi-bitw/cafr
- **Seurat 4.3.0** (Pinned dependency; provides the scRNA-seq data structure and downstream DEG analysis (FindAllMarkers) on attractor-defined clusters) — https://github.com/LingyiC/CASCC

## Examples

```
fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized", attr.raw = NULL, exponent.max = 10, exponent.min = 2)
```

## Evaluation signals

- CASCC::checkDependencies() completes without error, confirming Seurat 4.3.0 and cafr are installed
- fs.res is accepted by predictClusterK(data, fs.res, method = 'ward.D2', index = 'silhouette') without errors
- Feature-selected UMAP (res$res.predictK$adata) shows clusters that each correspond to a specific attractor (e.g. eight distinct clusters in the PDAC_peng_2k example)
- Downstream clustering from the selected features yields non-empty res$mainType.output$clusteringResults for each input dataset
- On benchmark datasets (e.g. Tabula Sapiens, 5,000–30,000 cells), clusters scored by ARI with aricode improve or match the paper's reported method comparison

## Limitations

- SIMLR fails with out-of-memory errors on large Tabula Sapiens datasets (5,000–30,000 cells), and memory-heavy attractor scans may face similar scaling limits; scanSeeds.adaptive exposes mc.cores for parallelization
- Feature selection assumes inputDataType 'well-normalized'; the exponent search is bounded by exponent.min = 2 and exponent.max = 10
- Attractor output may need manual refinement via attractorModify (percentage, overlapN) or collapseAttractorList (NumTopFeature, outputLength, removeDominant) parameters

## Evidence

- [methods] CASCC feature selection: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized""
- [methods] adaptive attractor seeding: "attr.res <- findAttractor.adaptive(Data_PDAC_peng_2k, "AIF1")"
- [methods] downstream clustering of selected features: "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette""
- [readme] method rationale: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [intro] installation dependencies: "remotes::install_version("Seurat", "4.3.0") # dependency"
