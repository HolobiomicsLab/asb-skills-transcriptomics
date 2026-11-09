---
name: single-cell-rna-seq-count-preprocessing
description: Use when raw single-cell RNA-seq count matrices (e.g., AnnData .h5ad
  from a multi-batch atlas like the NEURIPS bone marrow challenge) must be cleaned,
  normalized to 10000 counts per cell, log1p-transformed, and reduced to highly variable
  genes before topic-model or VAE training.
when_to_use_negative:
- The count matrix has already been depth-normalized or log-transformed — re-applying
  normalize_total and log1p would distort the counts
- The data is single-cell ATAC-seq or multimodal accessibility data, which follows
  a different MIRA preprocessing workflow rather than gene-expression filtering thresholds
- Only raw FASTQ files are available — alignment and quantification to a count matrix
  must happen before this skill applies
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Implements the filtering (cells < 400 counts, genes < 30 counts), normalize_total
    (target_sum=10000), log1p transformation, and highly variable gene selection (min
    dispersion 0.3) steps
  repo: https://scanpy.readthedocs.io/en/stable/
- name: anndata
  role: Container format for the count matrices, metadata, and batch covariates passed
    to MIRA; MIRA is built on top of scanpy and anndata
  repo: https://anndata.readthedocs.io/en/latest/
- name: MIRA / CODAL
  role: Downstream consumer of the preprocessed counts; its expression topic model
    expects filtered HVGs and deviance-residual-transformed raw counts with batch
    covariates
  repo: github.com/cistrome/MIRA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:63b9266c51167334296125153f2e371d0255c366afd905cd1d139f2b0268ee12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIRA
  - MIRA / CODAL
  - PyTorch
  - Pyro
  - anndata
  - scanpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-rna-seq-count-preprocessing

## Summary

Scanpy-based quality control and normalization of raw scRNA-seq count matrices (e.g., the NEURIPS 2021 bone marrow multi-batch dataset) prior to CODAL/MIRA topic-model training: cell and gene count filtering, depth normalization, log1p transformation, and highly variable gene selection.

## When to use

Use when raw single-cell RNA-seq count matrices (e.g., AnnData .h5ad from a multi-batch atlas like the NEURIPS bone marrow challenge) must be cleaned, normalized to 10000 counts per cell, log1p-transformed, and reduced to highly variable genes before topic-model or VAE training.

## When NOT to use

- The count matrix has already been depth-normalized or log-transformed — re-applying normalize_total and log1p would distort the counts
- The data is single-cell ATAC-seq or multimodal accessibility data, which follows a different MIRA preprocessing workflow rather than gene-expression filtering thresholds
- Only raw FASTQ files are available — alignment and quantification to a count matrix must happen before this skill applies

## Inputs

- Raw scRNA-seq count matrix as an AnnData object (.h5ad), e.g., NEURIPS 2021 bone marrow data with batch labels (sequencing site, donor)
- Cell and gene metadata (batch of origin / sequencing site / donor identifiers used as technical covariates)

## Outputs

- Filtered and log-normalized AnnData object (cells >= 400 counts, genes >= 30 counts, normalize_total target_sum=10000, log1p)
- Highly variable gene subset (3500 genes for the NEURIPS bone marrow dataset at min dispersion 0.3)
- Deviance-residual-transformed expression matrix for CODAL/MIRA encoder input

## How to apply

Load raw counts into a scanpy/AnnData object and first filter out cells with fewer than 400 total counts and genes with fewer than 30 total counts, removing empty droplets and rare transcripts. Normalize cell read depths with scanpy's normalize_total using target_sum=10000, then log+1 transform the counts. Select highly variable genes with a minimum dispersion threshold of 0.3, which on the NEURIPS bone marrow data yields 3500 highly variable genes. If the counts feed a CODAL/MIRA expression topic model, additionally transform the raw counts to deviance residuals before the encoder, and keep the technical covariates (sequencing site s1-s4 and donor d1-d10 batch labels) attached to the AnnData object since downstream training uses them as batch proxies.

## Related tools

- **scanpy** (Implements the filtering (cells < 400 counts, genes < 30 counts), normalize_total (target_sum=10000), log1p transformation, and highly variable gene selection (min dispersion 0.3) steps) — https://scanpy.readthedocs.io/en/stable/
- **anndata** (Container format for the count matrices, metadata, and batch covariates passed to MIRA; MIRA is built on top of scanpy and anndata) — https://anndata.readthedocs.io/en/latest/
- **MIRA / CODAL** (Downstream consumer of the preprocessed counts; its expression topic model expects filtered HVGs and deviance-residual-transformed raw counts with batch covariates) — github.com/cistrome/MIRA

## Examples

```
import scanpy as sc; sc.pp.filter_cells(adata, min_counts=400); sc.pp.filter_genes(adata, min_counts=30); sc.pp.normalize_total(adata, target_sum=10000); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, min_disp=0.3)
```

## Evaluation signals

- Cell count after filtering: every retained cell has >= 400 total counts and every retained gene has >= 30 counts
- Per-cell size factors: total normalized expression equals 10000 per cell before log1p
- HVG count matches expectation (~3500 highly variable genes on the NEURIPS bone marrow data at min_disp=0.3)
- Batch covariates (sequencing site s1-s4, donor d1-d10) remain intact and aligned cell-by-cell in the filtered AnnData for downstream CODAL training
- No NaN/negative values in the deviance-residual matrix fed to the encoder

## Limitations

- Thresholds (400 counts/cell, 30 counts/gene, min dispersion 0.3, target_sum 10000) were tuned for the NEURIPS bone marrow dataset; other tissues or sequencing depths may need re-tuning
- The paper cautions that downstream batch-confounded cell states should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- The article does not report software version numbers or computational implementation details for the preprocessing steps

## Evidence

- [methods] Cell and gene QC filtering: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] Depth normalization: "cell read depths normalized using the normalize_total function with target_sum set to 10000"
- [methods] Log transform and HVG selection: "counts log+1 transformed, and highly variable genes found with a minimum dispersion threshold of 0.3"
- [methods] Deviance residuals for encoder input: "Raw expression counts are transformed using deviance residuals61 before being fed through the encoder model."
- [readme] MIRA/scanpy/anndata stack: "MIRA works on top of `Scanpy <https://scanpy.readthedocs.io/en/stable/>`_ and `Anndata <https://anndata.readthedocs.io/en/latest/>`_"
