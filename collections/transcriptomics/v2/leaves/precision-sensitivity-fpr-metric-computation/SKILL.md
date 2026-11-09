---
name: precision-sensitivity-fpr-metric-computation
description: Use when a lineage prediction matrix must be quantitatively scored against
  ground-truth barcode assignments, e.g. reproducing GEMLI's cutoff-50 precision 80%±15%,
  sensitivity 22%±12%, FPR <1% on GSE99915 MEF data.
when_to_use_negative:
- The cell population contains only distantly related cells, where GEMLI performs
  less robustly and cutoff-50 benchmark ranges from clonally related MEF data would
  not be meaningful
- The task is lineage identification over time spans of weeks or months (long-term
  tracing), which exceeds GEMLI's scope of about 1–5 cell divisions and requires other
  approaches
- No independent ground-truth barcode-to-cell assignments exist, so precision/sensitivity
  cannot be computed against a reference
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: Provides predict_lineages (repeated iterative clustering to produce the confidence-score
    prediction matrix) and test_lineage_prediction (test_lineages) to compute TP,
    FP, precision, and sensitivity against barcode ground truth
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which the GEMLI package is installed via devtools and
    the benchmarking functions are executed
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install GEMLI from GitHub with install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:a34fcaafad39f39497a787f4782e9e418dac74615a219e6057ff00697ce3dd05
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# precision-sensitivity-fpr-metric-computation

## Summary

Benchmark GEMLI's cell lineage predictions against barcode ground truth by computing TP, FP, precision, sensitivity, and false positive rate (FP/(FP+TN)) at a confidence cutoff of 50, then validating against the paper's reported ranges for the MEF dataset.

## When to use

Use when a lineage prediction matrix must be quantitatively scored against ground-truth barcode assignments, e.g. reproducing GEMLI's cutoff-50 precision 80%±15%, sensitivity 22%±12%, FPR <1% on GSE99915 MEF data.

## When NOT to use

- The cell population contains only distantly related cells, where GEMLI performs less robustly and cutoff-50 benchmark ranges from clonally related MEF data would not be meaningful
- The task is lineage identification over time spans of weeks or months (long-term tracing), which exceeds GEMLI's scope of about 1–5 cell divisions and requires other approaches
- No independent ground-truth barcode-to-cell assignments exist, so precision/sensitivity cannot be computed against a reference

## Inputs

- Normalized, QC-filtered gene expression matrix (rows = genes, columns = cells) from GSE99915 (Biddy et al.) mouse embryonic fibroblast scRNA-seq data
- Ground-truth barcode-to-cell assignments from the CellTag barcoding experiments
- GEMLI_items list with slots 'gene_expression' and 'barcodes' (values = barcode ID, names = cell IDs)
- GEMLI cell-vs-cell confidence-score prediction matrix from predict_lineages

## Outputs

- TP, FP, precision, and sensitivity values across confidence intervals from test_lineage_prediction
- False positive rate computed as FP/(FP+TN) at confidence cutoff 50
- Testing results table saved as gemli_mef_testing_results.csv

## How to apply

Assemble a GEMLI_items list with slots 'gene_expression' and 'barcodes' (values = barcode ID, names = cell IDs) from the QC'd, normalized GSE99915 MEF matrix. Run GEMLI's predict_lineages, which identifies lineages through repeated iterative clustering, to obtain the cell-vs-cell confidence-score prediction matrix. Call test_lineage_prediction (test_lineages) against the barcoding ground truth to compute TP, FP, precision, and sensitivity across confidence intervals, then extract metrics at confidence cutoff 50 and compute the false positive rate as FP/(FP+TN). Accept the run only if precision falls within 0.65–0.95, sensitivity within 0.10–0.34, and FPR within 0.00–0.0015, matching the paper's reported 0.07% ± 0.08% FPR, and save the testing table as gemli_mef_testing_results.csv.

## Related tools

- **GEMLI** (Provides predict_lineages (repeated iterative clustering to produce the confidence-score prediction matrix) and test_lineage_prediction (test_lineages) to compute TP, FP, precision, and sensitivity against barcode ground truth) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which the GEMLI package is installed via devtools and the benchmarking functions are executed) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install GEMLI from GitHub with install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(GEMLI); GEMLI_items <- list(gene_expression=gene_matrix, barcodes=barcode_ground_truth); predictions <- predict_lineages(GEMLI_items); results <- test_lineage_prediction(GEMLI_items)
```

## Evaluation signals

- Precision at confidence cutoff 50 falls within 0.65–0.95 (paper mean ± s.d.: 80% ± 15%)
- Sensitivity at cutoff 50 falls within 0.10–0.34 (paper mean ± s.d.: 22% ± 12%)
- False positive rate FP/(FP+TN) falls within 0.00–0.0015 (paper: 0.07% ± 0.08%), consistently under 1%
- The testing results table gemli_mef_testing_results.csv is produced and contains TP, FP, precision, and sensitivity across confidence intervals
- test_lineage_prediction accepts the same GEMLI_items structure (slots 'gene_expression' and 'barcodes') that was fed to predict_lineages

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1–5 cell divisions; long-term lineage tracing requires other approaches

## Evidence

- [methods] predict_lineages identifies lineages through repeated iterative clustering and test_lineages scores predictions against barcode ground truth: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] The test_lineage_prediction function validates predictions against ground truth: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [results] Reported cutoff-50 performance on the MEF dataset: 80%±15% precision, 22%±12% sensitivity, FPR consistently under 1%: [evidence span withheld — anchor status: not_found]
- [readme] GEMLI is installed from GitHub as an R package using devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [discussion] GEMLI performs less robustly on populations of only distantly related cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
