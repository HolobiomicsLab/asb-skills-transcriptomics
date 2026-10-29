---
name: single-cell-lineage-prediction
description: Use when you have a quality-controlled, normalized scRNA-seq gene expression
  matrix (genes x cells) and want to infer close lineage relationships (over ~1-5
  cell divisions) without dedicated lineage-tracing assays, ideally with a barcode
  dictionary available for validation.
when_to_use_negative:
- Cell populations contain only distantly related cells — GEMLI performs less robustly
  there
- Long-term lineage tracing over weeks or months is needed — gene expression memory
  covers only ~1-5 cell divisions and genetic-mark approaches are better suited
- Input is raw unprocessed reads or lacks a QC-filtered, normalized expression matrix
  — lineage confidence scores assume normalized scRNA-seq counts as input
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: GEMLI
  role: R package that predicts cell lineages from scRNA-seq via gene expression memory
    (predict_lineages, test_lineages, trim_network_to_size, visualize_as_network)
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install GEMLI from GitHub with install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which the GEMLI package functions are executed
  repo: https://github.com/UPSUTER/GEMLI
- name: igraph
  role: Library required for network visualization of predicted lineages via visualize_as_network
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:6b844f357e15a45a6bde0991cdb6f77a4a19c8ed7c4b1561bbdd96a3960c2a29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - igraph
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-lineage-prediction

## Summary

Predict small to medium-sized cell lineages solely from scRNA-seq data using GEMLI's gene expression memory approach, and validate predictions against an independent CRISPR barcoding ground truth by computing precision and sensitivity across confidence levels.

## When to use

Use when you have a quality-controlled, normalized scRNA-seq gene expression matrix (genes x cells) and want to infer close lineage relationships (over ~1-5 cell divisions) without dedicated lineage-tracing assays, ideally with a barcode dictionary available for validation.

## When NOT to use

- Cell populations contain only distantly related cells — GEMLI performs less robustly there
- Long-term lineage tracing over weeks or months is needed — gene expression memory covers only ~1-5 cell divisions and genetic-mark approaches are better suited
- Input is raw unprocessed reads or lacks a QC-filtered, normalized expression matrix — lineage confidence scores assume normalized scRNA-seq counts as input

## Inputs

- Quality-controlled, normalized gene expression matrix (rows = genes, columns = cells), e.g. from GSM5283486 and GSM5283491 (GSE173958)
- Ground-truth CRISPR barcode dictionary (names = cell IDs, values = barcode ID) for validation
- GEMLI_items list holding expression data and optional lineage ground truth

## Outputs

- All-cells-against-all-cells matrix of lineage confidence scores from predict_lineages
- testing_results table with TP, FP, precision and sensitivity for confidence intervals 0-100
- Lineage information (cell-to-lineage assignments) via prediction_to_lineage_information
- Network visualization of predicted lineages and a precision-sensitivity curve (e.g. precision_sensitivity_curve.png)

## How to apply

Install GEMLI from GitHub with devtools via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0') and load it with library(GEMLI). Assemble a GEMLI_items list containing the QC-passed, normalized expression matrix (rows = genes, columns = cells) and, if available, a ground-truth dictionary mapping cell IDs to barcode IDs. Run predict_lineages(GEMLI_items), which identifies lineages through repeated iterative clustering (~2-3 min) and yields an all-cells-against-all-cells matrix of lineage confidence scores. Validate with test_lineages (test_lineage_prediction) against the barcode ground truth to obtain TP, FP, precision and sensitivity for confidence intervals from 0 to 100; in the mouse pancreatic cancer lung metastasis benchmark (GSM5283486/GSM5283491, GSE173958), GEMLI reached 100% precision at confidence level 80 while assigning more cells to lineages than the barcoding ground truth. Optionally extract lineage information with prediction_to_lineage_information, visualize with visualize_as_network (requires igraph), trim to a maximum lineage size with trim_network_to_size (e.g. max_size=2, cutoff=50), and re-run test_lineages with plot_results=T to render the precision-sensitivity curve.

## Related tools

- **GEMLI** (R package that predicts cell lineages from scRNA-seq via gene expression memory (predict_lineages, test_lineages, trim_network_to_size, visualize_as_network)) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install GEMLI from GitHub with install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which the GEMLI package functions are executed) — https://github.com/UPSUTER/GEMLI
- **igraph** (Library required for network visualization of predicted lineages via visualize_as_network)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); predict_lineages(GEMLI_items); test_lineages(GEMLI_items, plot_results=T)
```

## Evaluation signals

- testing_results table shows precision = 1.0 (100%) at confidence level 80 against the CRISPR barcoding ground truth
- GEMLI assigns at least as many cells to lineages as the barcoding ground truth while maintaining precision
- Precision-sensitivity curve over confidence intervals 0-100 is generated and monotonically trades sensitivity for precision as the cutoff rises
- predict_lineages completes within ~2-3 minutes and returns a square all-cells-against-all-cells confidence-score matrix
- Trimming parameters (e.g. trim_network_to_size with max_size=2, cutoff=50) are previewed with suggest_network_trimming_to_size before altering GEMLI_items

## Limitations

- GEMLI will generally perform less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- Lineage inference based on gene expression memory is limited to lineages over about 1-5 cell divisions; long-term lineage tracing over weeks or months requires other approaches such as naturally occurring genetic marks

## Evidence

- [abstract] GEMLI identifies small to medium-sized lineages from scRNA-seq: "a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets"
- [methods] GEMLI install via devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [methods] predict_lineages runs iterative clustering: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] test lineage prediction validates against ground truth: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [discussion] GEMLI less robust for distantly related cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
- [readme] README installation instructions: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory"
