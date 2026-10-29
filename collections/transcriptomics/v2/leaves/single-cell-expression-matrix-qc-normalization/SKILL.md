---
name: single-cell-expression-matrix-qc-normalization
description: Use when a raw gene-by-cell count matrix (e.g. from the MEF barcoding
  dataset GSE99915 of Biddy et al.) must be QC'd and normalized into a rows=genes,
  columns=cells matrix before lineage prediction.
when_to_use_negative:
- The cell population contains only distantly related cells — GEMLI performs less
  robustly in such populations, so lineage prediction downstream of this QC is not
  the right approach
- The goal is long-term lineage tracing over time spans of weeks or months — GEMLI's
  scope is limited to lineages over about 1–5 cell divisions and other approaches
  (e.g. genetic marks) are needed
- The input is not a single-cell expression matrix (e.g. already-aggregated clone
  tables or bulk RNA-seq), since the GEMLI items format requires gene-by-cell data
  with named cell barcodes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package whose GEMLI items input format ('gene_expression' and 'barcodes'
    slots) defines the required output schema of this QC/normalization step; consumed
    downstream by predict_lineages
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI package from GitHub before the normalized matrix
    can be assembled and used
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which the expression matrix QC/normalization and GEMLI
    items assembly are performed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:31dd45234bb9ddacef2d2b87107ced5c7e756284f93e0f9449f56a59aaba67d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-expression-matrix-qc-normalization

## Summary

Quality-control and normalize a gene-by-cell scRNA-seq expression matrix and package it as a GEMLI items list (slots 'gene_expression' and 'barcodes') so that GEMLI's lineage prediction can be run downstream.

## When to use

Use when a raw gene-by-cell count matrix (e.g. from the MEF barcoding dataset GSE99915 of Biddy et al.) must be QC'd and normalized into a rows=genes, columns=cells matrix before lineage prediction.

## When NOT to use

- The cell population contains only distantly related cells — GEMLI performs less robustly in such populations, so lineage prediction downstream of this QC is not the right approach
- The goal is long-term lineage tracing over time spans of weeks or months — GEMLI's scope is limited to lineages over about 1–5 cell divisions and other approaches (e.g. genetic marks) are needed
- The input is not a single-cell expression matrix (e.g. already-aggregated clone tables or bulk RNA-seq), since the GEMLI items format requires gene-by-cell data with named cell barcodes

## Inputs

- scRNA-seq gene expression count matrix (rows = genes, columns = cells), e.g. from GEO accession GSE99915 (Biddy et al. mouse embryonic fibroblast data)
- Ground-truth barcode-to-cell assignment table for validation
- Cell barcode list from the 10x CellRanger filtered results (e.g. hf1.d15.barcodes.tsv)

## Outputs

- QC'd and normalized gene expression matrix (rows = genes, columns = cells)
- GEMLI items list with slots 'gene_expression' and 'barcodes' (values = barcode ID, names = cell IDs), ready for predict_lineages

## How to apply

Download the scRNA-seq dataset (e.g. GSE99915 from GEO) plus any ground-truth barcode-to-cell assignments, then QC and normalize the gene expression matrix so rows are genes and columns are cells. Assemble a GEMLI items list with slots 'gene_expression' and 'barcodes', where barcode values are the barcode IDs and names are the cell IDs, so GEMLI's functions can consume it directly. Install GEMLI from GitHub using devtools (install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")) and load it with library(GEMLI); dependencies install with the package. The rationale is that GEMLI predicts lineages via repeated iterative clustering on expression memory, so a consistently normalized matrix with correctly named cell barcodes is a prerequisite for reliable predict_lineages and test_lineage_prediction output.

## Related tools

- **GEMLI** (R package whose GEMLI items input format ('gene_expression' and 'barcodes' slots) defines the required output schema of this QC/normalization step; consumed downstream by predict_lineages) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI package from GitHub before the normalized matrix can be assembled and used) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which the expression matrix QC/normalization and GEMLI items assembly are performed)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI)
```

## Evaluation signals

- The assembled GEMLI items list has a 'gene_expression' matrix with rows = genes and columns = cells, and a 'barcodes' slot whose values are barcode IDs and names are cell IDs matching the matrix columns
- Downstream GEMLI lineage prediction on the QC'd MEF data reaches precision at confidence cutoff 50 within 0.65–0.95 and sensitivity within 0.10–0.34 of the paper's reported mean ± s.d.
- False positive rate (FP/(FP+TN)) at cutoff 50 falls within 0.00–0.0015 against the barcoding ground truth
- The barcodes retained in the GEMLI items are consistent with the filtered 10x CellRanger cell barcode list used for the dataset
- predict_lineages completes its repeated iterative clustering (roughly 2–3 minutes) without errors, indicating the matrix is properly normalized and keyed

## Limitations

- GEMLI will generally perform less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance
- GEMLI's scope is limited to lineages over about 1–5 cell divisions; long-term lineage tracing requires other approaches

## Evidence

- [other] GEMLI items assembly requirement: [evidence span withheld — anchor status: not_found]
- [methods] GEMLI input consumption: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] GEMLI install via devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [readme] GEMLI package purpose: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets"
