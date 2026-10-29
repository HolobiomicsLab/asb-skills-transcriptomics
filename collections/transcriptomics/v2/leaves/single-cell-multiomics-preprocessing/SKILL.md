---
name: single-cell-multiomics-preprocessing
description: 'Use when preparing paired single-cell multi-omics data (e.g. 10X Multiome,
  CITE-seq, TEA-seq) for integrative methods like Mowgli: raw count matrices per modality
  need QC, normalization, and feature selection.'
when_to_use_negative:
- Data is unpaired single-modality scRNA-seq where standard Scanpy-only preprocessing
  (without Muon multi-modality handling) suffices
- Input is already normalized and feature-selected (e.g. a preprocessed .h5ad), and
  re-filtering would double-normalize counts
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Muon
  role: Handles the multi-modality Muon object; provides filter_var, tfidf (ATAC),
    and clr (ADT) operations
  repo: https://github.com/cantinilab/mowgli
- name: Scanpy
  role: Provides normalize_total and log1p normalization of RNA counts and downstream
    analysis within the scverse ecosystem
  repo: https://github.com/cantinilab/mowgli
- name: Mowgli
  role: Downstream integration method consuming the preprocessed Muon object; implemented
    as a Python package in the scverse ecosystem
  repo: https://github.com/cantinilab/mowgli
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:f4a628b90added951be5e6f71c0d3b14631dd4686ed1405fca227fd5064a4a21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Mowgli
  - Muon
  - Python
  - Scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-multiomics-preprocessing

## Summary

Per-modality preprocessing of paired single-cell multi-omics data (RNA, ATAC, ADT) into a Muon object, following Mowgli's pipeline of QC filtering, modality-specific normalization, and feature selection prior to integration.

## When to use

Use when preparing paired single-cell multi-omics data (e.g. 10X Multiome, CITE-seq, TEA-seq) for integrative methods like Mowgli: raw count matrices per modality need QC, normalization, and feature selection.

## When NOT to use

- Data is unpaired single-modality scRNA-seq where standard Scanpy-only preprocessing (without Muon multi-modality handling) suffices
- Input is already normalized and feature-selected (e.g. a preprocessed .h5ad), and re-filtering would double-normalize counts

## Inputs

- Paired single-cell multi-omics raw count matrices (scRNA-seq, scATAC-seq, ADT/CITE-seq) loadable as a Muon object
- Per-cell QC metadata (mitochondrial gene proportion, expressed gene counts, total counts)

## Outputs

- Preprocessed Muon/AnnData object with per-modality normalized and feature-selected matrices ready for Mowgli integration
- Filtered cell and feature sets (post-QC)

## How to apply

Load the paired data as a Muon object with one modality per assay. Perform quality control filtering of cells on the proportion of mitochondrial gene expression, number of expressed genes, and total counts; filter genes and ATAC peaks on the number of cells expressing them using Muon's filter_var. Normalize RNA with Scanpy's normalize_total (cells summed to 10000) followed by log1p, use TF-IDF via Muon's tfidf for ATAC peaks, and Center Log Ratio via Muon's clr for ADT proteins. Select the top 2500 most variable genes for RNA (1500 for the Liu/scCAT-seq dataset) as the feature-selection step before downstream integration with Mowgli.

## Related tools

- **Muon** (Handles the multi-modality Muon object; provides filter_var, tfidf (ATAC), and clr (ADT) operations) — https://github.com/cantinilab/mowgli
- **Scanpy** (Provides normalize_total and log1p normalization of RNA counts and downstream analysis within the scverse ecosystem) — https://github.com/cantinilab/mowgli
- **Mowgli** (Downstream integration method consuming the preprocessed Muon object; implemented as a Python package in the scverse ecosystem) — https://github.com/cantinilab/mowgli

## Examples

```
import muon as mu; mdata = mu.read('multiome.h5mu'); mu.pp.filter_var(mdata['atac'], mdata['atac'].var['n_cells'] > 3); sc.pp.normalize_total(mdata['rna'], target_sum=10000); sc.pp.log1p(mdata['rna']); sc.pp.highly_variable_genes(mdata['rna'], n_top_genes=2500)
```

## Evaluation signals

- Cells passing QC have mitochondrial proportion, expressed-gene count, and total counts within applied thresholds
- RNA counts sum to 10000 per cell after normalize_total and are log1p-transformed (non-negative, log-scale)
- Feature matrix contains exactly the top 2500 most variable genes (or 1500 for the Liu/scCAT-seq dataset)
- ATAC peaks are TF-IDF normalized and ADT values are Center Log Ratio transformed, with no NaNs introduced by filter_var

## Limitations

- Feature-selection thresholds are dataset-specific (2500 genes generally, 1500 for the Liu dataset); applying 2500 blindly to smaller datasets may be inappropriate
- Normalization choices are modality-specific — using normalize_total on ATAC or ADT instead of TF-IDF/CLR would deviate from the validated pipeline

## Evidence

- [methods] Quality control filtering of cells was performed on the proportion of mitochondrial gene expression, the number of expressed genes, and the total number of counts: [evidence span withheld — anchor status: not_found]
- [methods] Cells were normalized to sum to 10000 (using Scanpy's normalize_total): [evidence span withheld — anchor status: not_found]
- [methods] The top 2500 most variable genes (1500 for the Liu dataset) were selected for downstream analysis: [evidence span withheld — anchor status: not_found]
- [methods] In OP Multiome, cells were normalized using TF-IDF (using Muon's tfidf): [evidence span withheld — anchor status: not_found]
- [results] Users can thus take advantage of scverse tools like Scanpy and Muon for preprocessing and downstream analysis: [evidence span withheld — anchor status: not_found]
