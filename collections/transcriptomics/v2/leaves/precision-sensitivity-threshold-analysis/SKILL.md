---
name: precision-sensitivity-threshold-analysis
description: Use when scRNA-seq lineage predictions from GEMLI (or a similar predictor)
  must be validated against an independent lineage-tracing ground truth, such as a
  CRISPR barcode dictionary, to find an optimal confidence cutoff.
when_to_use_negative:
- The scRNA-seq population contains only distantly related cells — GEMLI performs
  less robustly, so precision-sensitivity evaluation at low lineage sizes may be misleading
- Lineage relationships span weeks or months (long-term tracing) — GEMLI's scope is
  limited to lineages over about 1-5 cell divisions
- No independent lineage ground truth (e.g. CRISPR barcodes) exists for the dataset
  — test_lineages cannot compute precision or sensitivity without one
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: GEMLI
  role: Provides predict_lineages and test_lineages functions used to generate lineage
    confidence scores and evaluate precision/sensitivity across confidence levels
    0-100
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI R package from GitHub via install_github("UPSUTER/GEMLI",
    subdir="GEMLI_package_v0")
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Execution environment for the GEMLI package and its testing/ploting functions
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:d514e88bde764a1f86f798d6747ceb9a0f80abb4aa4a10f7f69b29cd01f98c39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# precision-sensitivity-threshold-analysis

## Summary

Evaluate GEMLI's predicted cell lineages against a CRISPR-barcoding ground truth across confidence levels 0-100, quantifying TP, FP, precision and sensitivity at each threshold and selecting the confidence level (e.g. 80) where precision peaks.

## When to use

Use when scRNA-seq lineage predictions from GEMLI (or a similar predictor) must be validated against an independent lineage-tracing ground truth, such as a CRISPR barcode dictionary, to find an optimal confidence cutoff.

## When NOT to use

- The scRNA-seq population contains only distantly related cells — GEMLI performs less robustly, so precision-sensitivity evaluation at low lineage sizes may be misleading
- Lineage relationships span weeks or months (long-term tracing) — GEMLI's scope is limited to lineages over about 1-5 cell divisions
- No independent lineage ground truth (e.g. CRISPR barcodes) exists for the dataset — test_lineages cannot compute precision or sensitivity without one

## Inputs

- GEMLI_items list holding a normalized gene expression matrix (rows = genes, columns = cells) from GSE173958 samples GSM5283486 and GSM5283491
- CRISPR barcode ground-truth dictionary (names = cell IDs, values = barcode ID) used as lineage labels

## Outputs

- testing_results table with TP, FP, precision and sensitivity for each confidence level 0-100
- precision-sensitivity curve image (precision_sensitivity_curve.png) when plot_results=T
- chosen confidence threshold (e.g. 80) at which precision is maximal

## How to apply

Assemble a GEMLI_items list containing the quality-controlled, normalized gene expression matrix (rows = genes, columns = cells) and the ground-truth barcode dictionary (names = cell IDs, values = barcode ID). Run predict_lineages(GEMLI_items) to build the all-cells-against-all-cells matrix of lineage confidence scores via repeated iterative clustering (2-3 min). Then run test_lineages(GEMLI_items) to compare predictions against the barcode ground truth, computing TP, FP, precision and sensitivity for every confidence level from 0 to 100. Inspect the returned testing_results table to identify the confidence level with highest precision — in the mouse pancreatic cancer lung metastasis dataset (GSM5283486 and GSM5283491, GSE173958), precision reached 1.0 (100%) at confidence level 80. Re-run test_lineages with plot_results=T to render the precision-sensitivity curve as precision_sensitivity_curve.png for reporting the trade-off.

## Related tools

- **GEMLI** (Provides predict_lineages and test_lineages functions used to generate lineage confidence scores and evaluate precision/sensitivity across confidence levels 0-100) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI R package from GitHub via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")) — https://github.com/UPSUTER/GEMLI
- **R** (Execution environment for the GEMLI package and its testing/ploting functions) — https://github.com/UPSUTER/GEMLI

## Examples

```
test_lineages(GEMLI_items, plot_results=T)
```

## Evaluation signals

- testing_results table shows precision = 1.0 (100%) at confidence level 80 for the mouse pancreatic cancer lung metastasis dataset
- Number of cells assigned to lineages by GEMLI at the chosen threshold is greater than the number of cells captured by the CRISPR barcoding ground truth
- Precision and sensitivity are reported for the full confidence range 0-100, not just a single cutoff
- Precision-sensitivity curve renders successfully as precision_sensitivity_curve.png when plot_results=T
- Input GEMLI_items gene expression matrix has rows = genes and columns = cells, matching the expected orientation

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance
- Lineage inference with GEMLI is limited to lineages over about 1-5 cell divisions; long-term lineage tracing may require other approaches

## Evidence

- [methods] predict_lineages over iterative clustering: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] test_lineages evaluation: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [readme] Installation of GEMLI: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`. If `devtools` is not installed yet, you can do so with `install.packages("devtools")`"
- [readme] GEMLI purpose: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory"
