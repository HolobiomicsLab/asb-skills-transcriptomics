---
name: lineage-prediction-validation-against-barcoding-ground-truth
description: Use when GEMLI (or another scRNA-seq lineage inference tool) has produced
  cell-vs-cell lineage predictions and an independent barcode-to-cell assignment table
  exists to quantify precision, sensitivity, and false positive rate.
when_to_use_negative:
- No independent lineage ground truth (barcode-to-cell assignments) is available,
  since test_lineage_prediction cannot compute TP/FP without it
- The cell population contains only distantly related cells — the article notes GEMLI
  performs less robustly there and validation metrics would not transfer
- The goal is long-term lineage tracing over weeks or months; GEMLI's scope is limited
  to lineages over about 1-5 cell divisions
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: GEMLI
  role: R package providing predict_lineages (iterative clustering to produce the
    confidence-score prediction matrix) and test_lineage_prediction/test_lineages
    (compare predictions against barcoding ground truth to compute TP, FP, precision,
    sensitivity)
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment; GEMLI is installed via devtools install_github and executed
    as an R library
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install GEMLI from GitHub (install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"))
  repo: https://github.com/UPSUTER/GEMLI
- name: xcalibr
  role: 'Optional upstream barcode extraction: processes short reads and extracts
    a table of counts matched to barcode index/tag sequences, supplying ground-truth
    barcode references'
  repo: https://github.com/NKI-GCF/xcalibr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:4a8cb7d265b98adb098dbd3c99929a0fc59b072ad5e3b9a51edf759dcf3dbac4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - xcalibr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lineage-prediction-validation-against-barcoding-ground-truth

## Summary

Validates GEMLI scRNA-seq lineage predictions by testing them against experimental barcoding ground truth (e.g. CellTag data from the Biddy et al. mouse embryonic fibroblast dataset GSE99915), computing true positives, false positives, precision, sensitivity, and false positive rate across confidence cutoffs.

## When to use

Use when GEMLI (or another scRNA-seq lineage inference tool) has produced cell-vs-cell lineage predictions and an independent barcode-to-cell assignment table exists to quantify precision, sensitivity, and false positive rate.

## When NOT to use

- No independent lineage ground truth (barcode-to-cell assignments) is available, since test_lineage_prediction cannot compute TP/FP without it
- The cell population contains only distantly related cells — the article notes GEMLI performs less robustly there and validation metrics would not transfer
- The goal is long-term lineage tracing over weeks or months; GEMLI's scope is limited to lineages over about 1-5 cell divisions

## Inputs

- QC'd and normalized gene expression matrix (rows = genes, columns = cells) from a lineage-barcoded scRNA-seq dataset (e.g. GSE99915, Biddy et al. mouse embryonic fibroblasts)
- Ground-truth barcode-to-cell assignment table (CellTag barcode IDs per cell, values = barcode ID, names = cell IDs)
- GEMLI prediction output: cell-vs-cell confidence-score matrix from predict_lineages

## Outputs

- Lineage testing results table (CSV, e.g. gemli_mef_testing_results.csv) with TP, FP, precision, sensitivity across confidence intervals
- False positive rate computed as FP/(FP+TN) at the chosen confidence cutoff (e.g. cutoff 50: precision ~0.80, sensitivity ~0.22, FPR ~0.0007)

## How to apply

Assemble a GEMLI items list with slots 'gene_expression' (QC'd, normalized gene-by-cell matrix) and 'barcodes' (values = barcode ID, names = cell IDs) from the barcoded MEF scRNA-seq dataset. Run GEMLI's predict_lineages function, which identifies lineages through repeated iterative clustering (~2-3 min), to obtain the cell-vs-cell confidence-score prediction matrix. Then call test_lineage_prediction (test_lineages) with the same GEMLI_items and the barcoding ground truth to compute TP, FP, precision, and sensitivity across confidence intervals. Extract metrics at the confidence cutoff of interest (50 in the reference study) and compute false positive rate as FP/(FP+TN). Accept the run if precision at cutoff 50 falls within 0.65–0.95, sensitivity within 0.10–0.34, and FPR within 0.00–0.0015, matching the paper's reported 80% (±15%) precision, 22% (±12%) sensitivity, and FPR consistently under 1%. Save the testing results table (e.g. gemli_mef_testing_results.csv) for downstream comparison.

## Related tools

- **GEMLI** (R package providing predict_lineages (iterative clustering to produce the confidence-score prediction matrix) and test_lineage_prediction/test_lineages (compare predictions against barcoding ground truth to compute TP, FP, precision, sensitivity)) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment; GEMLI is installed via devtools install_github and executed as an R library) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install GEMLI from GitHub (install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"))) — https://github.com/UPSUTER/GEMLI
- **xcalibr** (Optional upstream barcode extraction: processes short reads and extracts a table of counts matched to barcode index/tag sequences, supplying ground-truth barcode references) — https://github.com/NKI-GCF/xcalibr

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); GEMLI_items$gene_expression <- exprs_matrix; GEMLI_items$barcodes <- barcode_assignments; predictions <- predict_lineages(GEMLI_items); test_results <- test_lineage_prediction(GEMLI_items)
```

## Evaluation signals

- Precision at confidence cutoff 50 falls within 0.65–0.95 (reference: 80% ± 15% S.D.)
- Sensitivity at confidence cutoff 50 falls within 0.10–0.34 (reference: 22% ± 12% S.D.)
- False positive rate, FP/(FP+TN), falls within 0.00–0.0015 (reference: 0.07% ± 0.08% S.D., consistently under 1%)
- Testing results table is produced with per-cutoff TP, FP, precision, and sensitivity columns and saved (e.g. gemli_mef_testing_results.csv)
- GEMLI_items input contains both 'gene_expression' and 'barcodes' slots with matching cell IDs before test_lineage_prediction runs

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage tracing requires other approaches
- Sensitivity is modest (~22% ± 12% at cutoff 50), so low sensitivity alone does not indicate a failed run if precision and FPR are in range

## Evidence

- [results] GEMLI small lineage prediction performance: [evidence span withheld — anchor status: not_found]
- [methods] test_lineages function: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [methods] predict_lineages clustering: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] GEMLI installation via devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
- [abstract] GEMLI robust lineage identification from scRNA-seq: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [readme] GSE99915 dataset availability: [evidence span withheld — anchor status: not_found]
