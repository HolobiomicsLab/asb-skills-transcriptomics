---
name: expression-skewness-gene-classification
description: Use when a barcode-annotated (lineage-labeled) scRNA-seq matrix must
  be screened for genes with unusually stable expression across related cells, and
  each hit classified as quantitative or qualitative memory.
when_to_use_negative:
- The scRNA-seq dataset has no lineage annotations (barcodes or otherwise) — lineage-mean
  CV2 cannot be computed against a lineage-level null without them
- The population contains only distantly related cells, where GEMLI's gene-expression-memory
  approach performs less robustly
- The goal is long-term lineage tracing over weeks or months — expression-memory signals
  cover roughly 1–5 cell divisions
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GEMLI
  role: R package providing the gene-expression-memory framework used to call genes
    with high expression memory on predicted or annotated cell lineages
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Implementation language for the CV2/null-permutation/skewness classification
    workflow and CSV export
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:8fbf4f59ea61533ba23eb6c75c3aa97c1a01f8eb6dc1d33fc84ac5087c273d22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Expression-skewness gene classification

## Summary

Identifies 'memory genes' whose expression is stably maintained across cell divisions in lineage-annotated scRNA-seq, by comparing per-gene lineage-mean CV2 against a lineage-size-matched null, then splits them into quantitative (skewness < 3) and qualitative (skewness ≥ 3) memory classes.

## When to use

Use when a barcode-annotated (lineage-labeled) scRNA-seq matrix must be screened for genes with unusually stable expression across related cells, and each hit classified as quantitative or qualitative memory.

## When NOT to use

- The scRNA-seq dataset has no lineage annotations (barcodes or otherwise) — lineage-mean CV2 cannot be computed against a lineage-level null without them
- The population contains only distantly related cells, where GEMLI's gene-expression-memory approach performs less robustly
- The goal is long-term lineage tracing over weeks or months — expression-memory signals cover roughly 1–5 cell divisions

## Inputs

- Quality-controlled, normalized gene-by-cell expression matrix from a barcoded scRNA-seq dataset (e.g. GSE99915 MEF data)
- Barcode-based lineage annotations mapping cell IDs to barcode/lineage IDs

## Outputs

- Gene-level table (e.g. memory_genes_MEF.csv) with columns for CV2, p-value, skewness, and memory class (quantitative/qualitative)
- Counts of total, quantitative, and qualitative memory genes

## How to apply

Download a barcoded scRNA-seq dataset (e.g. MEF CellTag data, GEO accession GSE99915), load the quality-controlled, normalized gene expression matrix and the barcode-based lineage annotations mapping cell IDs to barcode IDs. Aggregate expression per barcoded lineage and, for every gene, compute the coefficient of variation squared (CV2) of lineage-mean expression across lineages. Build a null CV2 distribution per gene using random lineage-size-matched cell groupings, and call memory genes whose lineage-mean CV2 is significantly higher than random (p ≤ 0.05). Compute the skewness of each significant memory gene's expression distribution and assign the quantitative class when skewness < 3 (widely/highly expressed, conserved within lineages) or the qualitative class when skewness ≥ 3 (highly variable, expressed in few lineages). Write a gene-level table with CV2, p-value, skewness, and class columns and report total, quantitative, and qualitative counts, validating against expected ranges.

## Related tools

- **GEMLI** (R package providing the gene-expression-memory framework used to call genes with high expression memory on predicted or annotated cell lineages) — https://github.com/UPSUTER/GEMLI
- **R** (Implementation language for the CV2/null-permutation/skewness classification workflow and CSV export) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI)
```

## Evaluation signals

- Called memory genes have lineage-mean CV2 significantly above the lineage-size-matched random null at p ≤ 0.05
- Every gene in the output carries a non-missing skewness value and a class label consistent with the threshold (quantitative if skewness < 3, qualitative if skewness ≥ 3)
- Resulting counts fall within reported ranges: total 3668 ± 1131; quantitative 2286 ± 890; qualitative 1382 ± 413
- Quantitative memory genes are predominantly widely/highly expressed, while qualitative memory genes are highly variable and expressed in only few lineages

## Limitations

- Memory-gene detection depends on lineage annotations of sufficient depth; lineages over about 1–5 cell divisions are the supported scope, and long-term tracing needs other approaches
- Large variability in growth rates between lineages may decrease performance
- Performance is less robust in cell populations that contain only distantly related cells

## Evidence

- [methods] CV2 significance call: [evidence span withheld — anchor status: not_found]
- [results] skewness class assignment: [evidence span withheld — anchor status: not_found]
- [results] validation ranges: [evidence span withheld — anchor status: not_found]
- [readme] GEMLI purpose: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory on the predicted cell"
- [readme] biological basis: "some genes show varying gene expression across cell lineages that is stable over multiple cell generations"
