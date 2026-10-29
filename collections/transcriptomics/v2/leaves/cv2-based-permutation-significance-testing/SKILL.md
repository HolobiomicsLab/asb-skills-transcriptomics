---
name: cv2-based-permutation-significance-testing
description: Use when a quality-controlled, normalized scRNA-seq matrix with barcode-based
  lineage annotations is available and you need to call genes with expression conserved
  across cell divisions versus a permutation null.
when_to_use_negative:
- The scRNA-seq dataset lacks barcode- or otherwise-derived lineage annotations, so
  no true lineage-size-matched null can be constructed
- Cells are only distantly related (no small/medium lineages over ~1-5 cell divisions),
  where lineage-inference approaches like GEMLI perform less robustly
- The goal is lineage prediction itself rather than gene-expression-memory testing;
  use GEMLI's predict_lineages workflow instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Analysis environment used to aggregate lineage expression, compute CV2, run
    the size-matched permutation null, compute skewness, and write the memory-gene
    table
  repo: https://github.com/UPSUTER/GEMLI
- name: GEMLI
  role: R package implementing gene-expression-memory computations for calling genes
    with high gene expression memory on predicted or annotated lineages
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:3eefe4d69ab50293872d427562adeaa7e1f00a525bf133c7c33348f13003fc47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cv2-based-permutation-significance-testing

## Summary

Identifies 'memory genes' whose expression is unusually stable within lineage-annotated scRNA-seq data by testing each gene's squared coefficient of variation (CV2) of lineage-mean expression against a null built from random lineage-size-matched cell groupings.

## When to use

Use when a quality-controlled, normalized scRNA-seq matrix with barcode-based lineage annotations is available and you need to call genes with expression conserved across cell divisions versus a permutation null.

## When NOT to use

- The scRNA-seq dataset lacks barcode- or otherwise-derived lineage annotations, so no true lineage-size-matched null can be constructed
- Cells are only distantly related (no small/medium lineages over ~1-5 cell divisions), where lineage-inference approaches like GEMLI perform less robustly
- The goal is lineage prediction itself rather than gene-expression-memory testing; use GEMLI's predict_lineages workflow instead

## Inputs

- Quality-controlled, normalized gene-by-cell expression matrix from barcoded scRNA-seq (e.g. MEF dataset from GEO accession GSE99915)
- Barcode-based lineage annotations mapping cell IDs to barcode IDs
- Permutation parameters (number of random lineage-size-matched groupings; significance threshold p ≤ 0.05)

## Outputs

- Gene-level table (e.g. memory_genes_MEF.csv) with columns: CV2, p-value, skewness, memory class
- Counts of total memory genes (expected ~3668 ± 1131), quantitative memory genes (~2286 ± 890), and qualitative memory genes (~1382 ± 413)

## How to apply

Aggregate the normalized expression matrix per barcoded lineage (cell IDs mapped to barcode IDs) and compute, for every gene, the CV2 of lineage-mean expression across lineages. Build a per-gene null CV2 distribution by repeatedly regrouping cells into random lineages matched to the true lineage sizes, then call memory genes whose observed lineage-mean CV2 is significantly lower (more stable) than random at p ≤ 0.05. Classify each significant memory gene by the skewness of its expression distribution: skewness < 3 marks quantitative memory genes (widely/highly expressed, conserved within lineages), while skewness ≥ 3 marks qualitative memory genes (highly variable, expressed in few lineages). Write a gene-level table with CV2, p-value, skewness, and class columns (e.g. memory_genes_MEF.csv) and report total, quantitative, and qualitative counts for validation against the published ranges.

## Related tools

- **R** (Analysis environment used to aggregate lineage expression, compute CV2, run the size-matched permutation null, compute skewness, and write the memory-gene table) — https://github.com/UPSUTER/GEMLI
- **GEMLI** (R package implementing gene-expression-memory computations for calling genes with high gene expression memory on predicted or annotated lineages) — https://github.com/UPSUTER/GEMLI

## Examples

```
Rscript -e 'library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")'
```

## Evaluation signals

- Total memory gene count falls within the reported range 3668 ± 1131
- Quantitative memory gene count falls within 2286 ± 890 and qualitative within 1382 ± 413
- Every called memory gene has lineage-mean CV2 significant against the size-matched permutation null at p ≤ 0.05
- Class assignment is consistent: skewness < 3 for all quantitative genes and skewness ≥ 3 for all qualitative genes
- Output table contains complete CV2, p-value, skewness, and class columns for all tested genes

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease performance
- Applicable to lineages over about 1-5 cell divisions; longer-term lineage tracing requires other approaches

## Evidence

- [methods] CV2 computation on lineage means: [evidence span withheld — anchor status: not_found]
- [methods] Permutation null and p-value threshold: [evidence span withheld — anchor status: not_found]
- [methods] Skewness-based classification: [evidence span withheld — anchor status: not_found]
- [results] Published memory-gene counts: [evidence span withheld — anchor status: not_found]
- [readme] GEMLI package purpose: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory on the predicted cell"
