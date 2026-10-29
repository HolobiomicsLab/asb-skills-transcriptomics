---
name: low-expression-gene-prefiltering
description: Use when a raw gene-by-cell FPKM/TPM expression matrix contains genes
  expressed (>5 FPKM/TPM) in fewer than two cells, and genes must be filtered before
  z-score normalization and clustering.
when_to_use_negative:
- Input is already a filtered, normalized feature matrix or a cell-by-gene count table
  in formats like .h5ad rather than a raw FPKM/TPM gene-by-cell matrix
- The goal is to filter low-quality or contaminated cells rather than genes — that
  is handled by filterLowQualityCells and filterContaminatedCells, not prefilterGenes
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 package whose prefilterGenes and expr.minimum functions perform the gene-level
    expression filtering and minimum-value flooring
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:9630b6b2b924cf85dbc09d47a7c09336b5c1466afb8620a3c5d5a581bcc2bc0f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# low-expression-gene-prefiltering

## Summary

Removes non- or low-expressed genes from an scRNA-Seq expression matrix using SINCERA's prefilterGenes function, then floors residual FPKM/TPM values with expr.minimum, as the gene-level half of SINCERA's preprocessing stage.

## When to use

Use when a raw gene-by-cell FPKM/TPM expression matrix contains genes expressed (>5 FPKM/TPM) in fewer than two cells, and genes must be filtered before z-score normalization and clustering.

## When NOT to use

- Input is already a filtered, normalized feature matrix or a cell-by-gene count table in formats like .h5ad rather than a raw FPKM/TPM gene-by-cell matrix
- The goal is to filter low-quality or contaminated cells rather than genes — that is handled by filterLowQualityCells and filterContaminatedCells, not prefilterGenes

## Inputs

- SINCERA S4 object created by construct from a gene-by-cell FPKM/TPM expression matrix (exprfile: rows = genes, columns = cells) and a single-column sample table (samplefile)
- Expression matrix already passed through filterLowQualityCells (and optionally filterContaminatedCells)

## Outputs

- Expression matrix with non- or low-expressed genes removed (genes expressed >5 FPKM/TPM in fewer than two cells dropped)
- Expression matrix with FPKM/TPM values <= 0.01 replaced by 0.01 (expr.minimum floor)
- Per-filter gene removal counts for the QC summary

## How to apply

After creating the SINCERA S4 object with construct and removing low-quality cells with filterLowQualityCells (min.expression = 1 FPKM/TPM, min.genes = 500), run prefilterGenes to drop genes expressed at more than 5 FPKM/TPM in fewer than two cells, since such genes carry no informative signal across the remaining cells. Then run expr.minimum with its default minimum of 0.01 FPKM/TPM to replace all FPKM/TPM values <= 0.01 with 0.01, which stabilizes downstream per-gene z-score normalization via normalization.zscore. Gene prefiltering is deliberately paired with cell prefiltering in the SINCERA pipeline — the order matters because gene expression abundancy is assessed over surviving cells only. Write the resulting filtered matrix (e.g. normalized_expression_matrix.tsv) and record how many genes each filter removed.

## Related tools

- **SINCERA** (R S4 package whose prefilterGenes and expr.minimum functions perform the gene-level expression filtering and minimum-value flooring) — github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA); sc <- prefilterGenes(sc); sc <- expr.minimum(sc, min.value = 0.01)
```

## Evaluation signals

- No gene in the filtered matrix exceeds 5 FPKM/TPM in fewer than two cells
- No FPKM/TPM value in the output matrix is below 0.01 (values <= 0.01 are replaced with 0.01)
- Only genes (not cells) changed relative to the input matrix; cell set matches the output of filterLowQualityCells/filterContaminatedCells
- The filtered matrix feeds a normalization.zscore per-gene z-score transformation without divide-by-near-zero artifacts
- QC summary accounts for every gene removed by each filter step

## Limitations

- Thresholds are FPKM/TPM-specific (prefilterGenes cutoff of >5 FPKM/TPM in two cells; expr.minimum floor of 0.01); raw UMI count matrices are not the assumed input scale
- Gene prefiltering runs after cell filtering — applying it on unfiltered cells can retain or remove the wrong genes
- SINCERA is designed for standard desktop/laptop computers and is implemented in R (GPL v3); very large datasets may not be its target scale

## Evidence
