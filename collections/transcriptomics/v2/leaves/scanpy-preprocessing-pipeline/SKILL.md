---
name: scanpy-preprocessing-pipeline
description: Use when raw count matrices from one or more scRNA-seq batches must be
  converted into a normalized, log-transformed AnnData object with top-2000 HVGs before
  training an integration model such as scDREAMER, scVI, or INSCT.
when_to_use_negative:
- Input is already normalized/log-transformed or preprocessed by another integration
  pipeline — reapplying normalize_total and HVG selection would double-process the
  data
- Analysis requires full-gene (non-HVG) expression, e.g., marker gene expression or
  subclustering of dendritic cell subtypes, since the pipeline discards genes outside
  the top 2000 HVGs
- Data modality is not scRNA-seq counts (e.g., multiomic or ATAC data), which the
  scDREAMER framework does not yet cover
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides AnnData import, scanly.pp.normalize_total normalization, log transformation,
    and scanpy.pp.highly_variable_genes (flavor='seurat') HVG selection feeding the
    integration models
- name: scDREAMER
  role: Downstream consumer of the preprocessed top-2000-HVG AnnData; trained with
    ADAM optimizer at batch size 128 to produce batch-mixed latent embeddings
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scVI
  role: Baseline integration method run on the same preprocessed data for fair benchmarking
- name: INSCT
  role: Baseline integration method consuming the same normalized AnnData with PCs
    and a batch vector
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:401d87fc05e8af555e5908ea14e0fbe6fa1321ec9e240332f8be79d64d64093b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - BBKNN
  - DESC
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# scanpy-preprocessing-pipeline

## Summary

Standard Scanpy preprocessing workflow used to prepare raw scRNA-seq count matrices for scDREAMER integration: import counts as AnnData, filter low-quality cells by mitochondrial gene counts, normalize_total + log-transform, and select the top 2000 highly variable genes (flavor='seurat').

## When to use

Use when raw count matrices from one or more scRNA-seq batches must be converted into a normalized, log-transformed AnnData object with top-2000 HVGs before training an integration model such as scDREAMER, scVI, or INSCT.

## When NOT to use

- Input is already normalized/log-transformed or preprocessed by another integration pipeline — reapplying normalize_total and HVG selection would double-process the data
- Analysis requires full-gene (non-HVG) expression, e.g., marker gene expression or subclustering of dendritic cell subtypes, since the pipeline discards genes outside the top 2000 HVGs
- Data modality is not scRNA-seq counts (e.g., multiomic or ATAC data), which the scDREAMER framework does not yet cover

## Inputs

- Raw count expression matrix (e.g., 10X/drop-seq counts) importable as a Scanpy AnnData object
- Processed benchmark datasets such as the human immune dataset (33,506 cells, ten batches) or lung atlas (32,472 cells) from the Figshare/Zenodo deposits
- Batch annotation vector stored in the AnnData object

## Outputs

- Filtered, normalized, log-transformed AnnData object
- AnnData subset to the top 2000 highly variable genes (flavor='seurat') used as scDREAMER input

## How to apply

Import the raw count expression matrices as a Scanpy AnnData object, then remove low-quality cells based on mitochondrial gene counts. Normalize the expression matrices with scanpy.pp.normalize_total and log-transform them. Select the top 2000 highly variable genes using scanpy.pp.highly_variable_genes with the flavor parameter set to 'seurat'; for multi-batch/multi-species inputs, keep only genes present across all batches so the same gene panel feeds every batch. The resulting AnnData (with the HVG subset) is the input to scDREAMER training (ADAM optimizer, batch size 128) and to all competing methods in the benchmark, ensuring a fair, identical preprocessing baseline.

## Related tools

- **Scanpy** (Provides AnnData import, scanly.pp.normalize_total normalization, log transformation, and scanpy.pp.highly_variable_genes (flavor='seurat') HVG selection feeding the integration models)
- **scDREAMER** (Downstream consumer of the preprocessed top-2000-HVG AnnData; trained with ADAM optimizer at batch size 128 to produce batch-mixed latent embeddings) — https://github.com/Zafar-Lab/scDREAMER
- **scVI** (Baseline integration method run on the same preprocessed data for fair benchmarking)
- **INSCT** (Baseline integration method consuming the same normalized AnnData with PCs and a batch vector)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('immune.h5ad'); sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat')
```

## Evaluation signals

- After preprocessing, the AnnData expression layer contains normalized, log-transformed values (no raw integer counts remain)
- The variable set passed to the integration model contains exactly the top 2000 highly variable genes selected with flavor='seurat'
- Low-quality cells flagged by mitochondrial gene counts are absent from the filtered object
- For multi-batch/cross-species inputs, the HVG panel contains only genes present across all batches, and all batches share the identical gene list
- Downstream Louvain clustering on scDREAMER embeddings from this preprocessing yields NMI/ARI and composite bio-conservation/batch-correction scores consistent with the published benchmark tables

## Limitations

- scDREAMER being a deep learning-based method may require fine-tuning of some parameters for extracting the best performance
- The current model assumes prior knowledge of the number of batches; batch information is not treated unsupervised
- For very large atlases (e.g., the healthy heart atlas), some downstream metrics such as kBET could not be computed for some methods due to computational limitations in handling large numbers of cells and batches

## Evidence

- [methods] Raw count expression matrices were imported as Scanpy AnnData object: "Raw count expression matrices were imported as Scanpy AnnData object"
- [methods] Normalize and log-transform: [evidence span withheld — anchor status: not_found]
- [methods] Top 2000 HVG selection: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER"
- [methods] Mitochondrial QC: "followed by the removal of low-quality cells based on the mitochondrial gene counts."
- [methods] Training hyperparameters after preprocessing: [evidence span withheld — anchor status: not_found]
- [readme] Conda environment setup for the preprocessing-to-integration pipeline: "Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml will install all the dependent packages and libraries."
