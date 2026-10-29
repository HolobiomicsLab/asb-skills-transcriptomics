---
name: tfidf-feature-transformation
description: Use when an unnormalized scATAC-seq gene activity matrix (h5ad) must
  be converted into the TF-IDF-normalized target input scBridge expects, prior to
  intersection with scRNA-seq common genes and integration training.
when_to_use_negative:
- When preprocessing the source scRNA-seq data, whose default strategy is 'Standard'
  rather than 'TFIDF'
- When input is still a peak-by-cell matrix — it must first be transformed into a
  gene activity matrix with tools such as Signac
- When the target data have already been TF-IDF-transformed and standardized; reapplying
  would distort the distribution fed to the encoder
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Integration framework whose --target_preprocess argument invokes the TFIDF
    (default) or Standard preprocessing of the scATAC-seq target data
  repo: https://github.com/XLearning-SCU/scBridge
- name: Signac
  role: Used upstream to transform scATAC-seq peak data into the scATAC-seq gene activity
    matrix before TF-IDF is applied
  repo: https://github.com/stuart-lab/signac
- name: Python
  role: Language in which scBridge is implemented (PyTorch 1.12.1 framework); anndata
    is used to read/write the h5ad activity matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:a09691d5ab98ae6299beec48b2d09aede452c0534e22ee023427b89d1d5cb99b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - Python
  - Signac
  - scBridge
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# tfidf-feature-transformation

## Summary

TF-IDF transformation of the scATAC-seq gene activity matrix into scBridge's default target (scATAC-seq) input representation, selected via the --target_preprocess argument, typically followed by standardization before feeding the encoder and classifier.

## When to use

Use when an unnormalized scATAC-seq gene activity matrix (h5ad) must be converted into the TF-IDF-normalized target input scBridge expects, prior to intersection with scRNA-seq common genes and integration training.

## When NOT to use

- When preprocessing the source scRNA-seq data, whose default strategy is 'Standard' rather than 'TFIDF'
- When input is still a peak-by-cell matrix — it must first be transformed into a gene activity matrix with tools such as Signac
- When the target data have already been TF-IDF-transformed and standardized; reapplying would distort the distribution fed to the encoder

## Inputs

- Raw scATAC-seq gene activity matrix as an AnnData (.h5ad) file
- scRNA-seq gene expression count matrix (h5ad) used to define the set of common genes

## Outputs

- Normalized scATAC-seq target matrix saved as target_processed.h5ad (AnnData format), TF-IDF-transformed and standardized over the common-gene feature space

## How to apply

Load the raw scATAC-seq gene activity matrix from its h5ad file with the anndata Python library, first having converted peak data into an activity matrix with a tool such as Signac if starting from peaks. Intersect the gene features with the scRNA-seq count matrix so only common genes are retained. Apply TF-IDF, which is scBridge's default for target data, configured through the --target_preprocess argument (values: 'TFIDF', default, or 'Standard'); note the source scRNA-seq data defaults to 'Standard' instead, so TF-IDF should only be applied to the target modality. After TF-IDF, standardize the matrix to zero means and unit variances as the Standard step of the preprocessing pipeline, then save the result as an AnnData h5ad (e.g., target_processed.h5ad). The rationale is that scATAC-seq gene activity data have a distribution and sparsity level vastly different from scRNA-seq, and TF-IDF rescaling mitigates this modality gap before scBridge's warm-up and reliable-cell-selection iterations.

## Related tools

- **scBridge** (Integration framework whose --target_preprocess argument invokes the TFIDF (default) or Standard preprocessing of the scATAC-seq target data) — https://github.com/XLearning-SCU/scBridge
- **Signac** (Used upstream to transform scATAC-seq peak data into the scATAC-seq gene activity matrix before TF-IDF is applied) — https://github.com/stuart-lab/signac
- **Python** (Language in which scBridge is implemented (PyTorch 1.12.1 framework); anndata is used to read/write the h5ad activity matrices)

## Evaluation signals

- Output is a readable AnnData h5ad file whose feature dimension equals the number of common genes retained after intersecting with the scRNA-seq matrix
- Output values reflect TF-IDF-then-standardize preprocessing: zero means and unit variances across features
- The transformation corresponds to scBridge's default target setting, i.e. target_preprocess='TFIDF' rather than 'Standard'
- The source (scRNA-seq) matrix was NOT TF-IDF-transformed — it retains the 'Standard' default source preprocessing

## Limitations

- Detailed discussion of TF-IDF preprocessing is deferred to Supplementary Note 3, which is not included in the provided document
- The scATAC-seq data must first be converted from peak-by-cell form into a gene activity matrix (e.g., via Signac) before this skill can be applied
- Genes must be intersected with the scRNA-seq count matrix to common genes prior to transformation, otherwise the encoder input dimension is invalid

## Evidence

- [methods] TFIDF default target preprocessing: "`--target_preprocess=["Standard", "TFIDF"]` (Default="TFIDF") the preprocessing strategy for target data"
- [readme] scBridge AnnData h5ad inputs: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
- [readme] gene activity matrix via Signac: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [intro] omics distribution difference: "the data distribution and sparsity level are vastly different across omics15"
