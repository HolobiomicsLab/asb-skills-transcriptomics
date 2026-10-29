---
name: single-cell-expression-normalization
description: Use when a raw UMI count matrix (gene rows × cell columns) must be converted
  into a normalized, log1p-scaled expression matrix before running Cancer-Finder,
  CopyKAT, SCEVAN, CaSee, or ikarus on scRNA-seq data.
when_to_use_negative:
- 'Feeding CopyKAT: it requires the raw gene expression UMI count matrix, not a normalized/log1p
  matrix'
- Spatial transcriptomics spot data that must preserve spatial relationships — Cancer-Finder
  overlooks spatial relationships among spots, so spatial-aware normalization methods
  would be preferred for such analyses
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: Performs pp.normalize_total and log1p log-transformed global scaling on each
    validation and training dataset before Cancer-Finder inference
  repo: Patchouli-M/SequencingCancerFinder
- name: Seurat
  role: Provides NormalizeData as the first step of the log-transformed global-scaling
    normalization applied to the 74 TISCH training datasets
- name: Cancer-Finder
  role: Downstream consumer of the normalized log1p matrices; consumes 4572 variance-selected
    highly variable genes
  repo: Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Baseline that explicitly requires the raw (un-normalized) UMI count matrix
    as its only input
  repo: navinlabcode/copykat
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:a2edf7e472260571f6d198be848360c6db87fafd31d8aadff889464ddec6de36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - CaSee
  - Cancer-Finder
  - CopyKAT
  - SCANPY
  - SCEVAN
  - Seurat
  - ikarus
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-expression-normalization

## Summary

Log-transformed global scaling normalization of single-cell RNA-seq count matrices (SCANPY pp.normalize_total + log1p, or Seurat NormalizeData + log1p) to prepare gene-by-cell inputs for Cancer-Finder and baseline annotation tools.

## When to use

Use when a raw UMI count matrix (gene rows × cell columns) must be converted into a normalized, log1p-scaled expression matrix before running Cancer-Finder, CopyKAT, SCEVAN, CaSee, or ikarus on scRNA-seq data.

## When NOT to use

- Feeding CopyKAT: it requires the raw gene expression UMI count matrix, not a normalized/log1p matrix
- Spatial transcriptomics spot data that must preserve spatial relationships — Cancer-Finder overlooks spatial relationships among spots, so spatial-aware normalization methods would be preferred for such analyses

## Inputs

- Raw UMI count matrix (gene symbols in rows, cell barcodes in columns), e.g. from 10X cellranger output
- SCANPY AnnData objects for the 10 external validation datasets (gold-standard 10x PBMC and cancer cell lines, silver-standard datasets 3-10)
- 74 TISCH tumor microenvironment datasets used for Cancer-Finder training

## Outputs

- Log1p-normalized, globally scaled expression matrix per dataset ready for Cancer-Finder inference
- Variance-ranked highly variable gene set (4572 input features for scRNA-seq model training)
- Raw count matrices preserved for CopyKAT runs

## How to apply

For Cancer-Finder input, preprocess each dataset with SCANPY 1.9.1 pp.normalize_total followed by log1p log-transformed global scaling; the same log-transformed global-scaling recipe (Seurat 'NormalizeData' followed by log1p) is applied when building the 74 TISCH training datasets. After normalization, rank genes by variance across all cells and select highly variable genes as model inputs — 4572 features for scRNA-seq data feed the 4572→512-neuron bottleneck of the Cancer-Finder network. Baseline tools differ in their input requirement: CopyKAT must instead receive the RAW UMI count matrix (not normalized values), possibly generated from 10X cellranger output via Seurat Read10X. Batching datasets with >10,000 cells by donor is applied before running CopyKAT. The rationale is to make expression scales comparable across tissues and datasets while preserving the raw counts required by CNV-inference baselines.

## Related tools

- **SCANPY** (Performs pp.normalize_total and log1p log-transformed global scaling on each validation and training dataset before Cancer-Finder inference) — Patchouli-M/SequencingCancerFinder
- **Seurat** (Provides NormalizeData as the first step of the log-transformed global-scaling normalization applied to the 74 TISCH training datasets)
- **Cancer-Finder** (Downstream consumer of the normalized log1p matrices; consumes 4572 variance-selected highly variable genes) — Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Baseline that explicitly requires the raw (un-normalized) UMI count matrix as its only input) — navinlabcode/copykat

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata); sc.pp.log1p(adata)
```

## Evaluation signals

- Normalized matrices should reproduce the reported downstream performance: Cancer-Finder 98.30% accuracy on gold-standard datasets and 90.89% similarity on silver-standard datasets
- Internal validation across the 13 TISCH training tissues should reach ~95.16% average accuracy after the normalization + HVG-selection pipeline
- Check that the input to CopyKAT remains raw UMI counts while Cancer-Finder inputs are log1p-normalized
- The feature dimension after variance-based gene selection must equal 4572 genes for the scRNA-seq model
- Per-tissue 1:1 downsampling of malignant/non-malignant cells and 80/20 train/validation split should be verified before training on normalized matrices

## Limitations

- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set, so normalization errors on training labels propagate
- Prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)
- Cancer-Finder is not recommended for hematologic tumor data even with correct normalization

## Evidence

- [methods] SCANPY normalize_total + log1p preprocessing for Cancer-Finder: [evidence span withheld — anchor status: not_found]
- [methods] Seurat-based log-transformed global scaling on TISCH training data: [evidence span withheld — anchor status: not_found]
- [readme] CopyKAT requires raw count input: [evidence span withheld — anchor status: external_doc]
