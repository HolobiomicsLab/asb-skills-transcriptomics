---
name: single-cell-rnaseq-normalization
description: Use when an annotated scRNA-seq count matrix must be converted to a comparable
  expression scale before gene selection or cell type classification, and the protocol
  (full-length vs UMI) dictates TPM+log1p or size-factor-10,000 normalization.
when_to_use_negative:
- Input is already log-normalized or provided as a normalized expression matrix (e.g.
  TPM-log values) — re-normalizing would distort likelihood-based classification
- Bulk RNA-seq or other non-single-cell count data where per-cell size-factor normalization
  semantics do not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: Consumes the normalized expression matrix for entropy-based gene selection
    (SelectGene), multinomial model training (Learn), and classification (Bet)
  repo: github.com/PaulingLiu/scibet
- name: Seurat v3
  role: Source of the default UMI preprocessing scheme (size-factor 10,000, add one,
    log normalization) adopted for UMI count normalization
- name: Python
  role: Used with scikit-learn/imbalanced-learn in the benchmarking pipeline that
    applies the normalization scheme
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:3b1000a5de61b7799dee19aee7935979e8b44d159bafb2c81806f378d059ba7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomoloGene
  - Python
  - SciBet
  - Seurat v3
  - Tabula Muris
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-rnaseq-normalization

## Summary

Technology-dependent preprocessing of scRNA-seq matrices as defined in the SciBet paper: full-length read data are TPM-normalized, pseudo-value one added, and log-transformed, while UMI data use Seurat v3-style size-factor 10,000, add-one, log normalization before any feature selection or classification.

## When to use

Use when an annotated scRNA-seq count matrix must be converted to a comparable expression scale before gene selection or cell type classification, and the protocol (full-length vs UMI) dictates TPM+log1p or size-factor-10,000 normalization.

## When NOT to use

- Input is already log-normalized or provided as a normalized expression matrix (e.g. TPM-log values) — re-normalizing would distort likelihood-based classification
- Bulk RNA-seq or other non-single-cell count data where per-cell size-factor normalization semantics do not apply

## Inputs

- Full-length scRNA-seq read count matrix (genes x cells)
- UMI scRNA-seq count matrix (genes x cells)
- Protocol/metadata flag indicating full-length vs UMI data

## Outputs

- Log-normalized expression matrix (TPM, +1, log for full-length; size-factor 10,000, +1, log for UMI)
- Normalized matrix suitable as input to SelectGene/Learn/Bet

## How to apply

First determine the sequencing protocol of the input matrix. For full-length read count data, compute Transcript Per Million (TPM), add a pseudo value of one to handle zero values, and perform log normalization. For UMI count data, normalize each cell with a size factor of 10,000 (counts scaled so total per cell equals 10,000), add one, and log-transform. This normalization is a prerequisite step upstream of SciBet's SelectGene (top 500 ΔS genes) and Learn (Laplace-smoothed multinomial model), so query and reference matrices must be normalized identically to keep expression vectors comparable. In cross-species settings, restrict to one-to-one HomoloGene orthologues before normalizing so the training and test matrices share gene space.

## Related tools

- **SciBet** (Consumes the normalized expression matrix for entropy-based gene selection (SelectGene), multinomial model training (Learn), and classification (Bet)) — github.com/PaulingLiu/scibet
- **Seurat v3** (Source of the default UMI preprocessing scheme (size-factor 10,000, add one, log normalization) adopted for UMI count normalization)
- **Python** (Used with scikit-learn/imbalanced-learn in the benchmarking pipeline that applies the normalization scheme)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- After log normalization, all expression values are strictly positive (pseudo-count one added) for full-length data
- For UMI data, each cell's pre-log total equals exactly 10,000 after size-factor scaling
- Reference (e.g. Tabula Muris) and query (e.g. human pancreas) matrices are normalized with the same scheme and restricted to intersecting orthologue genes
- Downstream SelectGene returns the default top 500 informative genes and Bet assigns cell types via multinomial MLE without scale mismatches
- Hold-out validation (stratified 7:3 split, 50 repeats) accuracy remains in the high range reported for SciBet, indicating normalization was applied consistently

## Limitations

- Two distinct schemes are required depending on technology — applying the TPM scheme to UMI data (or vice versa) is not validated in the paper
- The paper does not report explicit software versions (Seurat v3 minor version, scikit-learn, kallisto) or random seeds for the repeated normalization/classification splits
- Normalization must be applied after gene-space restriction in cross-species analysis; converting gene names after normalization with different gene panels can break comparability

## Evidence

- [discussion] TPM + log-normalization for full-length read count data: "we calculated Transcript Per Million (TPM)20, added pseudo value one to handle 0 values and performed log-normalization"
- [discussion] Seurat v3 UMI preprocessing: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [discussion] Technology-dependent defaults: "For unique molecular identifier (UMI) data, we applied the widely-used pre-processing methods proposed by Seurat v37 with default parameters"
- [methods] Downstream consumption of normalized matrix: "- `SelectGene(expr, k, r)`"
- [readme] Install command for the tool consuming normalized data: "devtools::install_github("PaulingLiu/scibet")"
