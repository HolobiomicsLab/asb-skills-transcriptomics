---
name: gene-expression-matrix-preprocessing
description: Use when a raw scRNA-seq expression matrix (genes x cells, count- or
  FPKM-like values) needs cleaning before dropout analysis, imputation, or clustering,
  and value range/gene sparsity must be normalized first.
when_to_use_negative:
- The matrix is already log-transformed/normalized and contains no extreme value range
  (range <= 100), so re-transforming would distort the data.
- The goal is single-cell analysis unrelated to zero-entry classification or imputation,
  where removing lowly expressed genes could discard biologically meaningful near-silent
  genes.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scISR
  role: Package whose preprocessing stage performs log2 rescaling and low-expression
    gene filtering before hypergeometric dropout testing
  repo: https://github.com/duct317/scISR
- name: R
  role: Implementation language for the scISR preprocessing and downstream testing
    steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:c3ebe6780270f1e22846fdc36e6b6eee44eff7e1e058c34a421a7f5e96678b69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R
  - scISR
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-expression-matrix-preprocessing

## Summary

Preprocess a raw scRNA-seq expression matrix before imputation/classification of zero entries by log2-transforming large-range count data and filtering genes with near-zero expression, as done in scISR prior to hypergeometric dropout testing.

## When to use

Use when a raw scRNA-seq expression matrix (genes x cells, count- or FPKM-like values) needs cleaning before dropout analysis, imputation, or clustering, and value range/gene sparsity must be normalized first.

## When NOT to use

- The matrix is already log-transformed/normalized and contains no extreme value range (range <= 100), so re-transforming would distort the data.
- The goal is single-cell analysis unrelated to zero-entry classification or imputation, where removing lowly expressed genes could discard biologically meaningful near-silent genes.

## Inputs

- Raw scRNA-seq expression matrix (genes x cells), e.g. count/FPKM/RPKM/TPM values such as the Goolam dataset matrix

## Outputs

- Preprocessed expression matrix with genes expressed in <2 cells or <1% non-zero entries removed and values log2-transformed when the original range exceeds 100

## How to apply

Start from the raw expression matrix of the scRNA-seq dataset. First check the value range: if it is larger than 100, rescale the data with a log transformation using base 2, which stabilizes heavily skewed count/FPKM scales. Then remove two categories of genes: (i) genes expressed in fewer than two cells, and (ii) genes that have less than one percent non-zero-valued entries, since these carry too little signal to inform dropout classification or regression-based imputation. The output is a filtered, optionally log2-transformed matrix that feeds scISR's hypergeometric testing module (and downstream subspace-regression imputation). Rationale: near-all-zero genes and extreme value ranges distort the per-gene zero tallies used by the phyper-based under/over-representation tests.

## Related tools

- **scISR** (Package whose preprocessing stage performs log2 rescaling and low-expression gene filtering before hypergeometric dropout testing) — https://github.com/duct317/scISR
- **R** (Implementation language for the scISR preprocessing and downstream testing steps)

## Examples

```
data('Goolam'); raw <- Goolam$data; raw_filtered <- raw[rowSums(raw != 0) > 0, ]; imputed <- scISR(data = raw)
```

## Evaluation signals

- No retained gene is expressed in fewer than two cells.
- No retained gene has less than one percent non-zero-valued entries.
- Value range of the raw matrix was checked; log2 transformation applied only when the range exceeded 100.
- Downstream hypergeometric per-gene zero tallies are computed on the filtered matrix, so all-zero genes do not appear in gene_partition.csv.
- Running scISR's imputation on the preprocessed matrix (e.g. Goolam) yields a valid imputed matrix suitable for PCA + k-means clustering with ARI evaluation.

## Limitations

- Filtering removes genes expressed in fewer than two cells, which could drop rare cell-type-specific transcripts from downstream analysis.
- The log2 transformation rule (range > 100) is a fixed heuristic tied to scISR's implementation; datasets already normalized (TPM/CPM) may or may not trigger it.
- The article does not state versions for the scISR software or the R stats package, limiting exact reproducibility.

## Evidence

- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [readme] Load Goolam dataset: data('Goolam'); raw <- Goolam$data; label <- Goolam$label: [evidence span withheld — anchor status: external_doc]
