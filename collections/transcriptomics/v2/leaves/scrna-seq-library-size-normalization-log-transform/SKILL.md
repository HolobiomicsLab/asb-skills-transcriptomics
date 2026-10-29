---
name: scrna-seq-library-size-normalization-log-transform
description: Use when starting from raw cell-by-gene FACS scRNA-seq counts (Tabula
  Muris Figshare CSVs) that must be filtered, normalized by library size, log-transformed,
  and reduced to 50 principal components before embedding evaluation.
when_to_use_negative:
- Input is a scATAC-seq counts-by-peaks matrix (the 10x PBMC data in this paper instead
  uses TSS-enrichment filtering and peak/cell count cutoffs like >=200 peaks and >10
  cells per peak)
- The dataset is not normalized or the goal is quality analysis of an entire unnormalized
  pipeline — the paper notes EMBEDR's null-generation process has not been extended
  to non-normalized datasets
- Data are already preprocessed (e.g. an existing PCA or embedding) and only require
  the DR-quality assessment step rather than re-normalization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EMBEDR
  role: Downstream consumer of the preprocessed 50-PC matrix; its empirical hypothesis
    testing over data/null marginal-resampling embeddings defines the required normalization
    state of the input
  repo: github.com/ejohnson643/EMBEDR
- name: numpy
  role: Loads the raw counts CSV into numeric arrays and supports library-size normalization
    and log transformation
  repo: github.com/ejohnson643/EMBEDR
- name: scikit-learn
  role: Provides PCA used to reduce the scaled HVG matrix to the first 50 principal
    components
- name: scipy
  role: Installation dependency of the EMBEDR package supporting sparse/numeric preprocessing
    operations
  repo: github.com/ejohnson643/EMBEDR
- name: czbiohub-sf/tabula-muris
  role: Source of the raw FACS counts CSVs and the data_ingest QC/normalization code
    that defines the filter thresholds
  repo: github.com/czbiohub/tabula-muris
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:66a56dfcc0052785413bea2ad1af114b0dfa287e505b396173a1687b06670795
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
  - czbiohub-sf/tabula-muris
  - numpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-library-size-normalization-log-transform

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw Tabula Muris FACS scRNA-seq count matrices (e.g. Diaphragm-counts.csv) by quality filtering, per-cell library-size normalization, log-transformation, and HVG selection/scaling to prepare a PCA-ready matrix (50 PCs) for downstream dimensionality-reduction quality assessment with EMBEDR.

## When to use

Use when starting from raw cell-by-gene FACS scRNA-seq counts (Tabula Muris Figshare CSVs) that must be filtered, normalized by library size, log-transformed, and reduced to 50 principal components before embedding evaluation.

## When NOT to use

- Input is a scATAC-seq counts-by-peaks matrix (the 10x PBMC data in this paper instead uses TSS-enrichment filtering and peak/cell count cutoffs like >=200 peaks and >10 cells per peak)
- The dataset is not normalized or the goal is quality analysis of an entire unnormalized pipeline — the paper notes EMBEDR's null-generation process has not been extended to non-normalized datasets
- Data are already preprocessed (e.g. an existing PCA or embedding) and only require the DR-quality assessment step rather than re-normalization

## Inputs

- Raw Tabula Muris FACS cell-by-gene counts CSV (e.g. 00_facs_raw_data/FACS/Diaphragm-counts.csv from the Figshare deposit)
- Quality-control thresholds from Tabula Muris Experimental Procedures (>=500 genes, >=50,000 reads, spike-in/ribosomal/Rn45s fraction caps)
- Number of highly variable genes and number of principal components to retain (50)

## Outputs

- Cell-by-gene normalized, log-transformed count matrix restricted to highly variable genes
- Centered/scaled cell-by-50 principal-component matrix saved as preprocessed_pca50.npz
- Set of retained cells satisfying gene-count and read-count thresholds

## How to apply

Load the raw FACS counts CSV (e.g. 00_facs_raw_data/FACS/Diaphragm-counts.csv from the Tabula Muris Figshare deposit) into a cell-by-gene count matrix with pandas/numpy. Apply cell quality-control thresholds per the Tabula Muris Experimental Procedures: retain cells with >=500 detected genes and >=50,000 total reads, and cap the fraction of reads from spike-ins, ribosomal genes, and Rn45s. Normalize each cell by library size and log-transform the counts, then restrict to highly variable genes and center/scale the values. Finally run scikit-learn PCA on the scaled matrix, retaining the first 50 principal components (cell-by-50 matrix), which is the input format EMBEDR expects for its data/null embedding comparisons.

## Related tools

- **EMBEDR** (Downstream consumer of the preprocessed 50-PC matrix; its empirical hypothesis testing over data/null marginal-resampling embeddings defines the required normalization state of the input) — github.com/ejohnson643/EMBEDR
- **numpy** (Loads the raw counts CSV into numeric arrays and supports library-size normalization and log transformation) — github.com/ejohnson643/EMBEDR
- **scikit-learn** (Provides PCA used to reduce the scaled HVG matrix to the first 50 principal components)
- **scipy** (Installation dependency of the EMBEDR package supporting sparse/numeric preprocessing operations) — github.com/ejohnson643/EMBEDR
- **czbiohub-sf/tabula-muris** (Source of the raw FACS counts CSVs and the data_ingest QC/normalization code that defines the filter thresholds) — github.com/czbiohub/tabula-muris

## Examples

```
df = pd.read_csv("../data/tabula-muris/00_facs_raw_data/FACS/Diaphragm-counts.csv"); counts = df.values.astype(float); norm = np.log1p(counts / counts.sum(axis=1, keepdims=True))
```

## Evaluation signals

- preprocessed_pca50.npz exists, loads as a numeric matrix, and has exactly 50 columns (one per retained principal component)
- Every retained cell passes the QC thresholds: >=500 detected genes and >=50,000 total reads
- Per-cell totals are consistent with library-size normalization (post-normalization count sums equalized before log transform)
- Fraction of reads from spike-ins, ribosomal genes, and Rn45s falls below the Experimental Procedures caps for all retained cells
- The PCA input matrix was restricted to highly variable genes and centered/scaled (zero mean per gene) before the 50-component reduction

## Limitations

- EMBEDR's null-generation process has not been extended to non-normalized datasets, so the pipeline must complete normalization before EMBEDR can validly assess embeddings
- The QC thresholds (>=500 genes, >=50,000 reads, spike-in/ribosomal/Rn45s caps) were defined for FACS-based full-length data; droplet-based 3'-end counting data have lower coverage and may require different cutoffs
- The number of principal components (50) is a fixed choice; the paper provides no automatic data-driven criterion for selecting this dimensionality in the preprocessing step

## Evidence

- [results] The Tabula Muris data preparation performs quality filtering on the raw counts, then transforms and scales the counts, and finally performs an initial dimensionality reduction step on the dataset.: [evidence span withheld — anchor status: not_found]

file_path = "../data/tabula-muris/"
data_name = f"00_facs_raw_data/FACS/{tissue.title()}-counts.csv""
- [readme] FACS-based full length transcript analysis, which provides higher sensitivity and coverage: [evidence span withheld — anchor status: external_doc]
