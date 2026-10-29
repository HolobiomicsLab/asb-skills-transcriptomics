---
name: cluster-number-estimation
description: Use when clustering scRNA-seq data where the true number of cell types
  is unknown, or when benchmarking how accurately clustering methods (e.g., CASCC
  vs. SC3) recover K against annotated ground-truth cell-type labels.
when_to_use_negative:
- K is already fixed a priori and you only need a final clustering — use kmeansCenterModify
  / run.kmeans.fixCenters directly instead of predictClusterK
- Data is raw counts that have not been normalized and log-transformed — CASCC expects
  'well-normalized' input for feature selection before K prediction
- The dataset is so large that dependency methods fail — e.g., SIMLR exited with out-of-memory
  errors on 5,000–30,000-cell Tabula Sapiens datasets and was excluded from the large-scale
  comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CASCC
  role: Provides predictClusterK (Ward.D2 + silhouette), CASCC.featureSelection, and
    kmeansCenterModify used to predict and realize the cluster number
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency implementing the unsupervised adaptive attractor algorithm underlying
    CASCC's co-expression features
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat
  role: Version-pinned dependency (4.3.0) for the CASCC workflow, e.g., FindAllMarkers
    on the feature-selected UMAP adata
- name: dplyr
  role: Used to assemble the 7 x 15 K_gen-vs-K_true scoring table in the number-of-cell-types
    benchmark
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:5cf927b3d5b5b2f1caaac85929a12abfac0726cbcd752274df3a7ee6c1c309d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - NbClust
  - R
  - Seurat
  - aricode
  - cafr
  - dplyr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# cluster-number-estimation

## Summary

Estimate the number of cell types (K) in scRNA-seq data by running CASCC's predictClusterK over attractor-based co-expression features (Ward.D2 hierarchical linkage with silhouette index), then quantify estimation accuracy against ground-truth cell-type labels using absolute log-modulus |log2(K_gen/K_true)| and deviation (K_gen - K_true) scores.

## When to use

Use when clustering scRNA-seq data where the true number of cell types is unknown, or when benchmarking how accurately clustering methods (e.g., CASCC vs. SC3) recover K against annotated ground-truth cell-type labels.

## When NOT to use

- K is already fixed a priori and you only need a final clustering — use kmeansCenterModify / run.kmeans.fixCenters directly instead of predictClusterK
- Data is raw counts that have not been normalized and log-transformed — CASCC expects 'well-normalized' input for feature selection before K prediction
- The dataset is so large that dependency methods fail — e.g., SIMLR exited with out-of-memory errors on 5,000–30,000-cell Tabula Sapiens datasets and was excluded from the large-scale comparison

## Inputs

- Normalized, log-transformed scRNA-seq expression matrix (inputDataType 'well-normalized', e.g., Tabula Sapiens organ datasets or benchmark datasets with cell-type annotations)
- CASCC feature-selection result fs.res from CASCC.featureSelection
- Ground-truth cell-type annotation table supplying K_true per dataset

## Outputs

- res.predictK from predictClusterK containing the predicted cluster number and clustering results
- res$mainType.output$clusteringResults after kmeansCenterModify
- Per method-dataset table of K_gen vs K_true with absolute log-modulus |log2(K_gen/K_true)| and deviation (K_gen - K_true) scores (e.g., supplementary_table_s5.csv)

## How to apply

First run CASCC.featureSelection with inputDataType = 'well-normalized' on normalized, log-transformed expression data to obtain attractor-based co-expression features. Then call predictClusterK(data, fs.res, method = 'ward.D2', index = 'silhouette', min.nc_fix = FALSE): hierarchical clustering on the feature-selected space with Ward.D2 linkage, selecting K via the silhouette index rather than a fixed cluster number. Finish the CASCC pipeline with kmeansCenterModify so the final k-means step uses modified attractor centers, and extract res$mainType.output$clusteringResults. To evaluate accuracy, extract the generated cluster number K_gen from the clustering output, take K_true from the ground-truth cell-type annotation table, and compute |log2(K_gen/K_true)| (absolute log-modulus) and (K_gen - K_true) (deviation) for every method-dataset pair, assembling a per-method table such as the paper's Supplementary Table S5 (7 methods x 15 datasets).

## Related tools

- **CASCC** (Provides predictClusterK (Ward.D2 + silhouette), CASCC.featureSelection, and kmeansCenterModify used to predict and realize the cluster number) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency implementing the unsupervised adaptive attractor algorithm underlying CASCC's co-expression features) — https://github.com/weiyi-bitw/cafr
- **Seurat** (Version-pinned dependency (4.3.0) for the CASCC workflow, e.g., FindAllMarkers on the feature-selected UMAP adata)
- **dplyr** (Used to assemble the 7 x 15 K_gen-vs-K_true scoring table in the number-of-cell-types benchmark)

## Examples

```
fs.res <- CASCC.featureSelection(data, inputDataType="well-normalized"); res.predictK <- predictClusterK(data, fs.res, method="ward.D2", index="silhouette", min.nc_fix=FALSE)
```

## Evaluation signals

- predictClusterK is called with method = 'ward.D2' and index = 'silhouette'; using a different linkage or index deviates from the published pipeline
- For every method-dataset pair, both K_gen and K_true are recorded so |log2(K_gen/K_true)| and (K_gen - K_true) are computable (105 pairs for 7 methods x 15 datasets)
- The assembled score table should reproduce the Supplementary Table S5 pattern: CASCC has the smallest absolute log-modulus and deviation, while SC3 overestimates cluster number on large benchmark datasets
- Downstream extraction uses res$mainType.output$clusteringResults (not the raw predictK output) as the final clustering
- Input expression data must be normalized and log-transformed ('well-normalized'); checkDependencies() confirms the pinned Seurat 4.3.0 / cafr environment

## Limitations

- K prediction quality depends on attractor-based feature selection; SC3 overestimates cluster number on large datasets while attractor-centered CASCC recovers K closer to the true value, so comparisons must use the same feature inputs
- On large Tabula Sapiens datasets (5,000–30,000 cells), some baseline methods such as SIMLR fail with out-of-memory errors and must be excluded from K-estimation comparisons
- The accuracy metrics are relative to the supplied ground-truth annotation; a mismatched cell-type label table invalidates log-modulus and deviation scores

## Evidence

- [methods] predictClusterK with Ward.D2 linkage and silhouette index selects K on attractor features: "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette", min.nc_fix = FALSE)"
- [methods] Feature selection with well-normalized input precedes cluster-number prediction: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized", attr.raw = NULL, exponent.max = 10, exponent.min = 2"
- [methods] Final clustering uses k-means with modified attractor centers: "res$mainType.output <- kmeansCenterModify(res)"
- [readme] README documents the pinned dependency environment required for reproducible K estimation: "remotes::install_version("Seurat", "4.3.0") # dependency
install_github("weiyi-bitw/cafr") # dependency
install_github("LingyiC/CASCC")
CASCC::checkDependencies()"
- [intro] Method rationale: co-expression features from an unsupervised adaptive attractor algorithm: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
