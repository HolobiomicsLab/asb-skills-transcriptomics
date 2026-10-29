---
name: scanpy-preprocessing-highly-variable-gene-selection
description: Use when raw scRNA-seq count matrices (imported as a Scanpy AnnData object)
  must be reduced to a compact, informative feature set before batch-integration methods
  such as scDREAMER, scVI, or INSCT are trained on multi-batch datasets.
when_to_use_negative:
- Input is already a reduced feature embedding or a pre-integrated matrix rather than
  raw counts in an AnnData object
- The dataset's biology is concentrated in lowly-expressed genes such that a 2000-gene
  'seurat' HVG selection would discard informative features (e.g., rare cell-type
  markers that must survive the selection for isolated-label F1 evaluation)
- The downstream workflow is supervised label transfer where full gene space or a
  different feature-selection recipe is required by the method's published configuration
tools:
- name: Scanpy
  role: Provides AnnData import, low-quality cell removal, normalize_total, log transformation,
    and highly_variable_genes (flavor='seurat') HVG selection
- name: scDREAMER
  role: Downstream integration model that consumes the top 2000 highly variable gene
    matrix as its training input
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: INSCT
  role: Baseline integration method operating on the same HVG-selected AnnData with
    PCs and a batch vector
  repo: https://github.com/lkmklsmn/insct
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:6ffdd42ebec01e211cac627ebeda06ae2b4f395c785eaa0b887765f2a070202b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - BBKNN
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# scanpy-preprocessing-highly-variable-gene-selection

## Summary

Selects the top 2000 highly variable genes from a Scanpy AnnData object using scanpy.pp.highly_variable_genes with flavor='seurat', after QC removal of low-quality cells (mitochondrial gene counts), normalize_total and log transformation. This reduced gene set is the input matrix used to train scDREAMER-style integration models.

## When to use

Use when raw scRNA-seq count matrices (imported as a Scanpy AnnData object) must be reduced to a compact, informative feature set before batch-integration methods such as scDREAMER, scVI, or INSCT are trained on multi-batch datasets.

## When NOT to use

- Input is already a reduced feature embedding or a pre-integrated matrix rather than raw counts in an AnnData object
- The dataset's biology is concentrated in lowly-expressed genes such that a 2000-gene 'seurat' HVG selection would discard informative features (e.g., rare cell-type markers that must survive the selection for isolated-label F1 evaluation)
- The downstream workflow is supervised label transfer where full gene space or a different feature-selection recipe is required by the method's published configuration

## Inputs

- Raw scRNA-seq count matrix as a Scanpy AnnData object (e.g., the processed human pancreas integration dataset from the Figshare deposit)
- Per-cell QC annotations such as mitochondrial gene counts for low-quality cell removal

## Outputs

- AnnData/log-transformed expression matrix restricted to the top 2000 highly variable genes (flavor='seurat')
- Normalized, log-transformed expression matrices feeding integration model training

## How to apply

First import raw count expression matrices as a Scanpy AnnData object and remove low-quality cells based on mitochondrial gene counts. Normalize the expression matrices using scanpy.pp.normalize_total and log-transform them. Then call scanpy.pp.highly_variable_genes with the flavor parameter set to 'seurat' to select the top 2000 highly variable genes, which are passed as the input to scDREAMER. The rationale is that restricting to 2000 variable genes removes noise features while preserving the biological signal (e.g., cell-type and batch structure in the 16,382-cell human pancreas task spanning nine sequencing protocols) needed for batch correction and bio-conservation metrics such as NMI and ARI. The selected matrix should retain the AnnData structure (cells x 2000 genes) so downstream Louvain clustering and metric computation operate on the same feature space across all compared methods.

## Related tools

- **Scanpy** (Provides AnnData import, low-quality cell removal, normalize_total, log transformation, and highly_variable_genes (flavor='seurat') HVG selection)
- **scDREAMER** (Downstream integration model that consumes the top 2000 highly variable gene matrix as its training input) — https://github.com/Zafar-Lab/scDREAMER
- **INSCT** (Baseline integration method operating on the same HVG-selected AnnData with PCs and a batch vector) — https://github.com/lkmklsmn/insct

## Examples

```
scanpy.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat')
```

## Evaluation signals

- AnnData object contains exactly 2000 genes after highly_variable_genes selection (n_top_genes=2000, flavor='seurat')
- Expression matrix is normalized and log-transformed before HVG selection (normalize_total applied first)
- Low-quality cells were removed based on mitochondrial gene counts prior to normalization
- The selected gene matrix supports successful downstream training of scDREAMER with the ADAM optimizer and batch size 128, and composite metrics (NMI, ARI, ASW, kBET, iLISI/cLISI) can be computed for all methods
- Same preprocessing and HVG recipe applied identically across the twelve compared methods so metric normalization (min-max across methods) is comparable

## Limitations

- scDREAMER being a deep learning-based method may require fine-tuning of some parameters for extracting the best performance, so the fixed 2000-gene HVG recipe may not be optimal for every dataset
- For the healthy heart atlas dataset, kBET could not be computed for some methods due to computational limitations in handling large number of cells and batches; very large datasets may stress the same preprocessing-to-training pipeline
- The current model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored

## Evidence

- [methods] HVG selection method and parameters: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”)"
- [methods] Preceding normalization step: [evidence span withheld — anchor status: not_found]
- [methods] QC step before normalization: "followed by the removal of low-quality cells based on the mitochondrial gene counts."
- [methods] Input format requirement: "Raw count expression matrices were imported as Scanpy AnnData object"
- [readme] Environment setup from README: "Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml will install all the dependent packages and libraries."
