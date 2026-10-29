---
name: iterative-clustering-lineage-inference
description: Use when you have a quality-controlled, normalized scRNA-seq gene expression
  matrix (genes x cells) and want to infer close lineage relationships (1-5 cell divisions)
  without dedicated lineage-tracing assays, ideally with a barcode dictionary available
  for validation.
when_to_use_negative:
- The cell population contains only distantly related cells, since GEMLI performs
  less robustly there
- Long-term lineage tracing over weeks or months is required; GEMLI is limited to
  lineages over about 1-5 cell divisions
- Input is raw unnormalized reads or unfiltered counts rather than a quality-controlled,
  normalized expression matrix
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: GEMLI
  role: Core R package providing predict_lineages, test_lineages, trim_network_to_size
    and related functions for iterative-clustering lineage inference
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install GEMLI from GitHub via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")
- name: igraph
  role: Required library for network visualization of predicted lineages via visualize_as_network
- name: R
  role: Runtime environment in which the GEMLI package and all lineage prediction/validation
    steps are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:fb8df6305c97dea461405832ae1ca768e441dd43b5edf852d761732fa4068ccf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - igraph
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# iterative-clustering-lineage-inference

## Summary

Predict small to medium-sized cell lineages (cells sharing a common ancestor) solely from scRNA-seq data using GEMLI, which exploits genes with stable heritable gene expression memory through repeated iterative clustering, and validate predictions against a CRISPR barcode ground truth.

## When to use

Use when you have a quality-controlled, normalized scRNA-seq gene expression matrix (genes x cells) and want to infer close lineage relationships (1-5 cell divisions) without dedicated lineage-tracing assays, ideally with a barcode dictionary available for validation.

## When NOT to use

- The cell population contains only distantly related cells, since GEMLI performs less robustly there
- Long-term lineage tracing over weeks or months is required; GEMLI is limited to lineages over about 1-5 cell divisions
- Input is raw unnormalized reads or unfiltered counts rather than a quality-controlled, normalized expression matrix

## Inputs

- Quality-controlled, normalized gene expression matrix from scRNA-seq (rows = genes, columns = cells), e.g. from GEO accession GSM5283486/GSM5283491 (GSE173958)
- Optional CRISPR barcode ground-truth dictionary (names = cell IDs, values = barcode ID) for validation
- GEMLI_items list combining the expression matrix and barcode dictionary

## Outputs

- All-cells-against-all-cells matrix of lineage confidence scores (0-100) from predict_lineages
- testing_results table with TP, FP, precision and sensitivity per confidence interval from test_lineages
- Precision-sensitivity curve image (e.g. precision_sensitivity_curve.png) when plot_results=T
- Lineage/cell-pair assignments at a chosen confidence cutoff, optionally trimmed to a maximum lineage size

## How to apply

Install the GEMLI R package from GitHub with devtools via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"), then library(GEMLI). Assemble a GEMLI_items list containing the normalized expression matrix (rows = genes, columns = cells; e.g. from GSM5283486 and GSM5283491 of GSE173958) and, optionally, a CRISPR barcode ground-truth dictionary (names = cell IDs, values = barcode ID). Run predict_lineages(GEMLI_items), which identifies lineages through repeated iterative clustering (~2-3 min) and returns an all-cells-against-all-cells matrix of lineage confidence scores from 0 to 100. If ground truth is available, run test_lineages(GEMLI_items) to obtain TP, FP, precision and sensitivity across confidence intervals 0-100; in the mouse pancreatic cancer lung metastasis dataset GEMLI reached 100% precision at confidence level 80 while assigning more cells to lineages than the CRISPR barcoding ground truth. Inspect predictions with test_lineages(plot_results=T) to render the precision-sensitivity curve, and refine or extract lineages with helper functions such as trim_network_to_size(GEMLI_items, max_size=2, cutoff=50) or prediction_to_lineage_information.

## Related tools

- **GEMLI** (Core R package providing predict_lineages, test_lineages, trim_network_to_size and related functions for iterative-clustering lineage inference) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install GEMLI from GitHub via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"))
- **igraph** (Required library for network visualization of predicted lineages via visualize_as_network)
- **R** (Runtime environment in which the GEMLI package and all lineage prediction/validation steps are executed)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); GEMLI_items <- list(expr = expr_matrix, barcodes = barcode_dict); results <- predict_lineages(GEMLI_items)
```

## Evaluation signals

- test_lineages against the CRISPR barcode ground truth yields precision = 1.0 (100%) at confidence level 80 in the mouse pancreatic cancer lung metastasis dataset
- predict_lineages returns a square all-cells-against-all-cells matrix of confidence scores bounded in 0-100 after ~2-3 min of repeated iterative clustering
- The precision-sensitivity curve (plot_results=T) renders without error and precision increases with confidence cutoff while sensitivity decreases
- At the chosen cutoff GEMLI assigns at least as many cells to lineages as the CRISPR barcoding ground truth (more cells assigned was observed)
- Installation succeeds via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0") and library(GEMLI) loads with dependencies installed

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage tracing may require other approaches based on naturally occurring genetic marks

## Evidence

- [abstract] GEMLI robustly identifies small to medium-sized lineages: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [methods] predict_lineages iterative clustering: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [readme] installation via devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [discussion] poor performance on distantly related cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
- [methods] network trimming parameters: [evidence span withheld — anchor status: not_found]
