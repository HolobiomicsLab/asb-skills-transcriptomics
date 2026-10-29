---
name: pca-dimensionality-reduction-preprocessing
description: Use when raw Tabula Muris FACS scRNA-seq count matrices (e.g. 00_facs_raw_data/FACS/{Tissue}-counts.csv
  from the Figshare deposit) must be converted into a numeric 50-PC feature matrix
  as input to embedding methods such as EMBEDR.
when_to_use_negative:
- Input is scATAC-seq data — the paper uses a different filter set for that modality
  (TSS enrichment, cells with <=200 peaks, peaks in <=10 cells), not the FACS gene/read-count
  thresholds
- Data is already normalized or already dimensionally reduced (e.g. an existing PCA
  or embedding matrix) — reapplying the raw-count filter/normalize/scale steps would
  distort the representation
- The dataset is non-normalized or not a Tabula Muris FACS count matrix and you intend
  to chain it directly into EMBEDR's null-generation process, which the paper notes
  has not been extended to non-normalized datasets
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the PCA implementation used to compute and retain the first 50 principal
    components of the scaled HVG matrix
- name: numpy
  role: Handles the cell-by-gene count matrix as numeric arrays and stores the PC
    matrix in .npz format
- name: EMBEDR
  role: Downstream consumer of the preprocessed cell-by-50 PC matrix for empirical-null
    embedding quality assessment
  repo: github.com/ejohnson643/EMBEDR
- name: czbiohub/tabula-muris data_ingest
  role: Reference code for ingesting the Tabula Muris FACS raw counts used in this
    pipeline
  repo: github.com/czbiohub/tabula-muris
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:f72ac711374a2edbdf335de9c018af294155efca89544023e691b419acb99d83
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - Python
  - czbiohub/tabula-muris data_ingest
  - numpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pca-dimensionality-reduction-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A scRNA-seq preprocessing pipeline that converts raw Tabula Muris FACS counts (e.g. Diaphragm-counts.csv) into a cell-by-50-principal-component matrix via quality filtering, library-size normalization with log transform, highly-variable-gene selection with centering/scaling, and scikit-learn PCA retaining 50 components.

## When to use

Use when raw Tabula Muris FACS scRNA-seq count matrices (e.g. 00_facs_raw_data/FACS/{Tissue}-counts.csv from the Figshare deposit) must be converted into a numeric 50-PC feature matrix as input to embedding methods such as EMBEDR.

## When NOT to use

- Input is scATAC-seq data — the paper uses a different filter set for that modality (TSS enrichment, cells with <=200 peaks, peaks in <=10 cells), not the FACS gene/read-count thresholds
- Data is already normalized or already dimensionally reduced (e.g. an existing PCA or embedding matrix) — reapplying the raw-count filter/normalize/scale steps would distort the representation
- The dataset is non-normalized or not a Tabula Muris FACS count matrix and you intend to chain it directly into EMBEDR's null-generation process, which the paper notes has not been extended to non-normalized datasets

## Inputs

- Raw Tabula Muris FACS counts CSV (cell-by-gene matrix, e.g. Diaphragm-counts.csv from the Figshare deposit)
- Experimental Procedures QC thresholds (>=500 genes, >=50,000 reads, spike-in/ribosomal/Rn45s fraction caps)

## Outputs

- preprocessed_pca50.npz — numeric cell-by-50 PC matrix
- Set of retained cells passing the gene-count and read-count thresholds

## How to apply

Load the raw FACS counts CSV into a cell-by-gene matrix with pandas/numpy, then apply the Experimental Procedures QC filters: retain cells with >=500 detected genes and >=50,000 total reads, and cap the fraction of reads from spike-ins, ribosomal genes, and Rn45s. Normalize each cell by library size and log-transform the counts. Restrict to highly variable genes, center and scale the values, then run scikit-learn PCA and retain the first 50 principal components. Save the result as preprocessed_pca50.npz so downstream EMBEDR analysis operates on a fixed 50-dimensional representation; this scaling-before-PCA step matches the paper's treatment of normalized, scaled data as DR input.

## Related tools

- **scikit-learn** (Provides the PCA implementation used to compute and retain the first 50 principal components of the scaled HVG matrix)
- **numpy** (Handles the cell-by-gene count matrix as numeric arrays and stores the PC matrix in .npz format)
- **EMBEDR** (Downstream consumer of the preprocessed cell-by-50 PC matrix for empirical-null embedding quality assessment) — github.com/ejohnson643/EMBEDR
- **czbiohub/tabula-muris data_ingest** (Reference code for ingesting the Tabula Muris FACS raw counts used in this pipeline) — github.com/czbiohub/tabula-muris

## Examples

```
from sklearn.decomposition import PCA; X_pca50 = PCA(n_components=50).fit_transform(X_hvg_scaled)
```

## Evaluation signals

- preprocessed_pca50.npz exists and loads as a numeric matrix
- The loaded matrix has exactly 50 columns (50 retained principal components)
- All retained cells satisfy >=500 detected genes and >=50,000 total reads, and respect the spike-in, ribosomal, and Rn45s fraction caps
- Values have been centered and scaled after HVG restriction (zero-mean columns in the PC-input matrix)
- The same cells' PCs reproduce deterministically given the same raw counts, filters, and HVG set

## Limitations

- The >=500-gene / >=50,000-read thresholds are Tabula Muris Experimental Procedures values; they do not transfer automatically to other tissues, chemistries, or scATAC-seq data
- EMBEDR's null-generation process has not been extended to non-normalized datasets, so this pipeline must produce scaled input before embedding
- Choice of highly variable genes and the fixed 50-PC cutoff are user-specified decisions that shape the downstream embedding (the paper stresses that DR quality depends on user-specified hyperparameters)

## Evidence

- [other] The Tabula Muris data preparation performs quality filtering on the raw counts, then transforms and scales the counts.: [evidence span withheld — anchor status: not_found]
- [other] Run principal component analysis (scikit-learn PCA) on the scaled matrix and retain the first 50 principal components.: [evidence span withheld — anchor status: not_found]
data_name = f"00_facs_raw_data/FACS/{tissue.title()}-counts.csv""
