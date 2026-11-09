---
name: geo-dataset-acquisition-and-loading
description: Use when a task requires downloading a specific GEO-accessioned single-cell
  RNA-seq dataset (e.g., GSE141259) and preparing raw count matrices as a Seurat object
  for embedding or hyperparameter-optimization analyses.
when_to_use_negative:
- Input is already a preprocessed/normalized Seurat object or an embedding — re-running
  acquisition and normalization would be redundant
- The target dataset is not publicly available on GEO (e.g., a bundled package dataset
  like the Hydra input_counts.rda), in which case load it directly instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Loads GEO count data into a Seurat object and performs NormalizeData, ScaleData,
    FindVariableFeatures, and RunPCA preprocessing
- name: R
  role: Execution environment for the download, loading, and Seurat preprocessing
    pipeline
- name: scDEED R package
  role: Downstream consumer of the loaded Seurat object for dubious-embedding detection
    and hyperparameter optimization
  repo: github.com/JSB-UCLA/scDEED
- name: devtools
  role: Installs the scDEED R package from GitHub prior to downstream analysis
  repo: github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:e05b05dc3e33d0d59392ec5e6409f029a74c3e03808d2d769f55d9fca4ac55a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - doParallel
  - scDEED R package
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# geo-dataset-acquisition-and-loading

## Summary

Acquire a public scRNA-seq dataset from GEO (e.g., the Alveolar dataset GSE141259, 29,297 cells) and load it into a Seurat object for downstream dimensionality-reduction analysis, using the Seurat preprocessing chain (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA).

## When to use

Use when a task requires downloading a specific GEO-accessioned single-cell RNA-seq dataset (e.g., GSE141259) and preparing raw count matrices as a Seurat object for embedding or hyperparameter-optimization analyses.

## When NOT to use

- Input is already a preprocessed/normalized Seurat object or an embedding — re-running acquisition and normalization would be redundant
- The target dataset is not publicly available on GEO (e.g., a bundled package dataset like the Hydra input_counts.rda), in which case load it directly instead

## Inputs

- GEO accession ID for a public scRNA-seq dataset (e.g., GSE141259)
- Raw gene-by-cell count matrix (input_counts) downloadable from the GEO repository

## Outputs

- Preprocessed Seurat object containing normalized and scaled expression data with PCA reduction computed
- Chosen input dimension K (number of PCs) determined from ElbowPlot

## How to apply

Download the raw count matrix for the target GEO accession (e.g., GSE141259 Alveolar scRNA-seq, 29,297 cells) and construct a Seurat object with CreateSeuratObject(input_counts). Apply the standard scDEED-compatible preprocessing chain in order: NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, since scDEED uses Seurat objects and by default takes the PCA space as the pre-embedding space. Choose the input dimension K (number of PCs) using Seurat::ElbowPlot before handing the object to embedding or scDEED functions. Note that scDEED's default input slot is 'scale.data', so the ScaleData step must precede any downstream embedding evaluation. Validate loading by confirming the expected cell count (29,297 for GSE141259) in the resulting Seurat object.

## Related tools

- **Seurat** (Loads GEO count data into a Seurat object and performs NormalizeData, ScaleData, FindVariableFeatures, and RunPCA preprocessing)
- **R** (Execution environment for the download, loading, and Seurat preprocessing pipeline)
- **scDEED R package** (Downstream consumer of the loaded Seurat object for dubious-embedding detection and hyperparameter optimization) — github.com/JSB-UCLA/scDEED
- **devtools** (Installs the scDEED R package from GitHub prior to downstream analysis) — github.com/JSB-UCLA/scDEED

## Examples

```
library(Seurat); data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data)
```

## Evaluation signals

- The loaded Seurat object contains the expected number of cells (e.g., 29,297 cells for the Alveolar GSE141259 dataset)
- The preprocessing chain (CreateSeuratObject → NormalizeData → ScaleData → FindVariableFeatures → RunPCA) completes without errors and the PCA reduction is present in the object
- The scale.data slot is populated, since scDEED defaults to slot='scale.data' as its PCA input space
- An ElbowPlot on the PCA reduction supports the chosen K before downstream embedding
- Downstream scDEED calls on the loaded object run with pre_embedding='pca' without input-space errors

## Limitations

- Seurat version differences can affect installation and behavior — the scDEED README notes version 5.2.1 was reportedly different from version 5.1.0; avoid updating all packages (especially Seurat) when installing via devtools
- The scDEED example pipeline is demonstrated on the Hydra dataset (input_counts.rda, 10000 randomly selected cells); reproducing results on other GEO datasets (e.g., Alveolar GSE141259) requires adapting the download step, and exact reproduced percentages may depend on preprocessing choices

## Evidence

- [other] GEO GSE141259 dataset: [evidence span withheld — anchor status: not_found]
- [readme] Seurat objects required: "First we will load the data. scDEED uses Seurat objects."
- [readme] Preprocessing chain from README: "data = CreateSeuratObject(input_counts)
data = NormalizeData(data)
data = ScaleData(data)
data = FindVariableFeatures(data)
data = RunPCA(data)"
- [readme] Slot default: "`slot`: (default = 'scale.data', the default slot in Seurat) the input space for PCA."
- [readme] ElbowPlot for K: "There are many ways the users can decide how many dimensions to use. A quick method may be to use an elbow plot."
