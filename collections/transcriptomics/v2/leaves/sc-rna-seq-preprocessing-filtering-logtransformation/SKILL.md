---
name: sc-rna-seq-preprocessing-filtering-logtransformation
description: Use when preparing raw or lightly processed scRNA-Seq count matrices
  (e.g. GEO downloads like GSE59739 CPM or GSE102962 TPM) before NMF dictionary learning,
  knowledge transfer, or downstream clustering such as SC3.
when_to_use_negative:
- The input is already cell- and gene-filtered and log-transformed — the scRNA scripts
  allow turning off these steps to avoid double-filtering or re-logging data
- Datasets share only a small set of transcripts, since after intersecting gene sets
  'a loss of (probably) vital information is inevitable' and the approach is deferred
  to future work
- A batch-effect-removal or imputation pre-processing (Seurat CCA or MAGIC) is the
  goal — these are separate additional pre-processing steps applied on top, not replacements
  for the three base steps
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scRNA (nicococo/scRNA)
  role: Python framework whose source/target command-line scripts implement the cell
    filter, gene filter, and log-transformation (each switchable off) before NMF transfer
    learning
  repo: https://github.com/nicococo/scRNA
- name: Seurat
  role: Optional additional pre-processing (batch effect removal, R script seurat_cca_hockley_usoskin.R)
    applied before the three base steps in robustness experiments
- name: MAGIC
  role: Optional additional pre-processing (imputation, matlab/magic_hockley_usoskin.m)
    applied to both datasets before the three base steps in robustness experiments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:9d93fe35d96eee1887e62ee703297d13a9276baabaf21ea51a8f7ee057cbd0a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - MAGIC
  - NMF
  - SC3
  - Seurat
  - scRNA (nicococo/scRNA)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sc-rna-seq-preprocessing-filtering-logtransformation

## Summary

Standard three-step pre-processing of scRNA-Seq count matrices (cell filter, gene filter, log-transformation after adding a pseudo-count of 1) used in Mieth et al. 2019 to prepare source and target datasets before NMF-based transfer learning and SC3 clustering.

## When to use

Use when preparing raw or lightly processed scRNA-Seq count matrices (e.g. GEO downloads like GSE59739 CPM or GSE102962 TPM) before NMF dictionary learning, knowledge transfer, or downstream clustering such as SC3.

## When NOT to use

- The input is already cell- and gene-filtered and log-transformed — the scRNA scripts allow turning off these steps to avoid double-filtering or re-logging data
- Datasets share only a small set of transcripts, since after intersecting gene sets 'a loss of (probably) vital information is inevitable' and the approach is deferred to future work
- A batch-effect-removal or imputation pre-processing (Seurat CCA or MAGIC) is the goal — these are separate additional pre-processing steps applied on top, not replacements for the three base steps

## Inputs

- Raw or normalized scRNA-Seq expression matrix with gene ids (e.g. TPM matrix for Hockley et al. 2018 GSE102962, 314 cells)
- Source dataset expression matrix (e.g. CPM matrix for Usoskin et al. 2014 GSE59739, 622 cells)
- Pre-processing thresholds xgenes and xexpression for the cell filter (from experiment scripts or supplementary material)

## Outputs

- Filtered and log-transformed expression matrices for source and target restricted to the common gene set (e.g. 4402 genes shared by Hockley and Usoskin)
- Gene ids and cell ids in .tsv format consistent with the filtered matrices, ready for NMF source-dictionary learning (scRNA-source.sh) and target clustering (scRNA-target.sh)

## How to apply

Apply three pre-processing steps to each dataset (source and target): (1) a cell filter removing all cells containing fewer than a gene-count threshold xgenes with expression above a threshold xexpression (concrete values are given in the experiment scripts, e.g. main_wrapper_hockley.py, and in Supplementary Sections 3.1 for Hockley/Usoskin); (2) a gene filter removing ubiquitous genes expressed in almost all cells and rare genes, intersecting source and target to the common gene set (4402 shared genes for Hockley–Usoskin); (3) log-transformation of the expression matrix after adding a pseudo-count of 1. The cell/gene filter and log transformation can be switched off via command-line flags (--no filters / no log transformation) when already processed data such as TPM or CPM matrices are used. Only after these steps should the target be encoded against the source dictionary Hsrc via NMF, so rationale is to harmonize scale and remove low-information genes/cells before dictionary transfer.

## Related tools

- **scRNA (nicococo/scRNA)** (Python framework whose source/target command-line scripts implement the cell filter, gene filter, and log-transformation (each switchable off) before NMF transfer learning) — https://github.com/nicococo/scRNA
- **Seurat** (Optional additional pre-processing (batch effect removal, R script seurat_cca_hockley_usoskin.R) applied before the three base steps in robustness experiments)
- **MAGIC** (Optional additional pre-processing (imputation, matlab/magic_hockley_usoskin.m) applied to both datasets before the three base steps in robustness experiments)

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git && scRNA-generate-data.sh  # or feed preprocessed matrices to scRNA-source.sh with filters/log-transformation disabled
```

## Evaluation signals

- After gene intersection, source and target matrices have the same gene dimension (4402 common genes for Hockley–Usoskin) and identical gene-id ordering
- Every retained cell passes the xgenes/xexpression threshold; log-transformed values are non-negative with zeros mapped to log(1)=0 due to the pseudo-count of 1
- The preprocessed files load into the scRNA workflow without the 'Intel MKL FATAL ERROR' class issues and NMF source step runs over the expected cluster range (default 6,7,8)
- Filter/log parameters match the paper's experiment scripts and Supplementary Section 3.1 (Hockley and Usoskin settings) so results reproduce the robustness table counts

## Limitations

- If source and target share only a small set of transcripts, restricting to common genes loses (probably) vital information; handling differing gene sets is unresolved and deferred to future work
- No software version numbers are stated for the pre-processing-adjacent tools (SC3, Seurat, MAGIC) in the methods, complicating exact reproduction
- The specific cell-filter thresholds are only in the scripts/supplementary material, not in the main text, so they must be retrieved from main_wrapper_hockley*.py or Supplementary Sections 3.1

## Evidence

- [methods] Three pre-processing steps: "Three steps for pre-processing scRNA-Seq data were applied"
- [methods] Cell filter definition: "Cell filter: Remove all cells containing fewer than xgenes genes with expression > xexpression"
- [methods] Gene filter definition: "Gene filter: Remove ubiquitous genes that are expressed in almost all cells"
- [methods] Log-transformation with pseudo-count: "Log-transformation: Log-transform the expression matrix after adding a pseudo-count of 1"
- [readme] Common gene set size: [evidence span withheld — anchor status: not_found]
- [readme] README on switching filters off: "Ie. we want to turn off the cell- and gene-filter as well as the log transformation."
