---
name: scrnaseq-preprocessing-quality-control
description: Use when raw scRNA-seq count data must be filtered (min/max genes per
  cell, min cells per gene, mitochondrial percentage, HVG dispersion) before single-cell
  drug-response prediction with scDEAL.
when_to_use_negative:
- Input is already a QC-filtered or processed AnnData (.h5ad) object from a prior
  scDEAL run — re-filtering would drop cells/genes against the benchmark settings
- Data are bulk RNA-seq cell-line profiles (ALL_expression.csv / ALL_label_binary_wf.csv)
  used to train bulkmodel.py, which do not undergo per-cell scRNA-seq QC
- Highly variable gene selection must keep all genes (var_genes_disp = None) because
  the study selects all genes for that dataset
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides the scanpy-package preprocessing utilities (scanpypip) used for cell/gene
    filtering and HVG selection prior to scDEAL prediction
- name: AnnData
  role: Container format for the QC-ed and predicted single-cell data; expected output
    is the AnnData (.h5ad) object applied by the scanpy package
- name: scDEAL
  role: Framework whose preprocessing step (min/max gene counts, min_c cells per gene,
    percent_mito, var_genes_disp) prepares each dataset for bulk-to-single-cell transfer
  repo: https://github.com/OSU-BMBL/scDEAL.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:c4dc5f498b619b0746e07da0c001842ea6ec335b086d66dedef82932dc607724
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - Anaconda
  - AnnData
  - Scanpy
  - conda
  - conda-pack
  - pip
  - scDEAL
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-preprocessing-quality-control

## Summary

Quality-control and preprocessing of drug-treated scRNA-seq count matrices (e.g. GSE117872, GSE112274, GSE140440, GSE149383, GSE110894) ahead of scDEAL transfer learning, using Scanpy-based gene/cell/mt-content filters and highly-variable-gene selection.

## When to use

Use when raw scRNA-seq count data must be filtered (min/max genes per cell, min cells per gene, mitochondrial percentage, HVG dispersion) before single-cell drug-response prediction with scDEAL.

## When NOT to use

- Input is already a QC-filtered or processed AnnData (.h5ad) object from a prior scDEAL run — re-filtering would drop cells/genes against the benchmark settings
- Data are bulk RNA-seq cell-line profiles (ALL_expression.csv / ALL_label_binary_wf.csv) used to train bulkmodel.py, which do not undergo per-cell scRNA-seq QC
- Highly variable gene selection must keep all genes (var_genes_disp = None) because the study selects all genes for that dataset

## Inputs

- Raw scRNA-seq count matrices from GEO accessions (e.g. GSE117872, GSE112274, GSE140440, GSE149383, GSE110894) unpacked from scDEAL.zip into the data/ directory
- QC parameter settings: min_n_genes, max_n_genes, min_g, min_c, percent_mito, var_genes_disp, PCA_dim

## Outputs

- Filtered scRNA-seq expression matrices ready for scmodel.py transfer learning
- AnnData (.h5ad) result objects with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) under save/adata

## How to apply

Run the Scanpy-based preprocessing utilities bundled with scDEAL (scanpypip) on each GEO dataset before bulk-to-single-cell transfer. Apply cell filters using the tool's thresholds: min_n_genes (default 0, cells with UMI counts >1), max_n_genes (default 20000), min_g (default 200 genes per cell >1), percent_mito (default 100), and gene filter min_c (default: keep genes expressed in at least 3 cells). Choose highly variable genes via var_genes_disp; if None, all genes are retained. Optionally reduce dimensionality with PCA_dim before training (0 means no PCA). The QC'd data then feeds bulkmodel.py/scmodel.py, whose AnnData outputs (.h5ad) with adata.obs['sens_label'] should be stored under save/adata for downstream metric computation (F1, AUROC, AP, precision, recall, AMI, ARI).

## Related tools

- **Scanpy** (Provides the scanpy-package preprocessing utilities (scanpypip) used for cell/gene filtering and HVG selection prior to scDEAL prediction)
- **AnnData** (Container format for the QC-ed and predicted single-cell data; expected output is the AnnData (.h5ad) object applied by the scanpy package)
- **scDEAL** (Framework whose preprocessing step (min/max gene counts, min_c cells per gene, percent_mito, var_genes_disp) prepares each dataset for bulk-to-single-cell transfer) — https://github.com/OSU-BMBL/scDEAL.git

## Evaluation signals

- Cell counts per dataset match expectations (e.g. GSE117872 = 548 cells, GSE112274 = 507, GSE140440 = 324, GSE149383 = 1496, GSE110894 = 1419) after filtering
- Every retained cell has at least min_g = 200 genes with counts >1 and at most max_n_genes = 20000
- Every retained gene is expressed in at least min_c = 3 cells
- Output is a valid AnnData (.h5ad) object loadable by scanpy, with adata.obs['sens_label'] containing 0/1 labels
- Downstream scmodel.py run on the QC-ed data reproduces benchmark scores (e.g. Data 1 F1 0.92 / AUROC 0.92 / AP 0.97)

## Limitations

- No software version numbers are stated for the tools or the conda environment, so exact reproducibility of the preprocessing behavior may vary
- Prediction accuracy can vary depending on the collection of bulk gene expression of cell lines; training data will be updated by integrating additional bulk-level databases
- A potential sample swap between Data 1 and 2 (HN120 and HN137, GSE117872) was pointed out by a recent study, affecting any QC on those datasets

## Evidence

- [methods] Minimum number of genes for a cell that have UMI counts >1 for filtering propose, default: 0: "Minimum number of genes for a cell that have UMI counts >1 for filtering propose, default: 0"
- [methods] Minimum number of cell that each gene express for filtering propose, default: 3: "Minimum number of cell that each gene express for filtering propose, default: 3"
- [readme] Expected output format of scDEAL is AnnData .h5ad applied by scanpy: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [readme] Dispersion of highly variable genes selection when pre-processing the data: "Dispersion of highly variable genes selection when pre-processing the data. If None, all genes will be selected"
