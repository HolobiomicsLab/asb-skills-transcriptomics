---
name: gene-expression-memory-based-lineage-prediction
description: Use when scRNA-seq data lacks lineage information and you need to infer
  small cell lineages, or when validating GEMLI lineage predictions against barcoding
  ground truth (e.g., GSE99915 MEF data) with precision/sensitivity/FPR metrics.
when_to_use_negative:
- Cell populations contain only distantly related cells, where GEMLI performs less
  robustly
- Long-term lineage tracing over weeks or months is needed; GEMLI is limited to lineages
  over about 1-5 cell divisions
- Large variability in growth rates between lineages is expected, which may decrease
  performance
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GEMLI
  role: R package providing predict_lineages, test_lineage_prediction, visualize_as_network,
    and trim_network_to_size for lineage inference from scRNA-seq
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install GEMLI from GitHub via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')
- name: R
  role: Runtime environment in which GEMLI is loaded and executed
- name: igraph
  role: Library required for network visualization of predicted lineages via visualize_as_network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:49aeae087459b5485c8bc9f2e2ef3eb87b064a2ac58939cf9cc13159239ddf57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - igraph
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-expression-memory-based-lineage-prediction

## Summary

GEMLI predicts small to medium-sized cell lineages (cells with a common ancestor over ~1-5 divisions) solely from scRNA-seq data by leveraging genes with stable expression over generations, and provides functions to test predictions against barcode-based ground truth.

## When to use

Use when scRNA-seq data lacks lineage information and you need to infer small cell lineages, or when validating GEMLI lineage predictions against barcoding ground truth (e.g., GSE99915 MEF data) with precision/sensitivity/FPR metrics.

## When NOT to use

- Cell populations contain only distantly related cells, where GEMLI performs less robustly
- Long-term lineage tracing over weeks or months is needed; GEMLI is limited to lineages over about 1-5 cell divisions
- Large variability in growth rates between lineages is expected, which may decrease performance

## Inputs

- QC'd and normalized scRNA-seq gene expression matrix (rows = genes, columns = cells) packaged in a GEMLI items list with slot 'gene_expression'
- Barcode-to-cell ground-truth assignments in a GEMLI items list with slot 'barcodes' (values = barcode ID, names = cell IDs), required for test_lineage_prediction
- Confidence cutoff value (e.g., 50) for lineage prediction and evaluation

## Outputs

- Cell-vs-cell confidence-score lineage prediction matrix from predict_lineages
- Testing results table with TP, FP, precision, sensitivity, and FPR across confidence cutoffs (e.g., gemli_mef_testing_results.csv)
- Lineage membership via prediction_to_lineage_information and network visualizations via visualize_as_network

## How to apply

Install GEMLI via devtools::install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0') and load with library(GEMLI). Assemble a GEMLI items list with slots 'gene_expression' (QC'd, normalized matrix, rows=genes, columns=cells) and 'barcodes' (values=barcode IDs, names=cell IDs) if ground truth is available. Run predict_lineages(GEMLI_items), which identifies lineages through repeated iterative clustering (~2-3 min) and returns a cell-vs-cell confidence-score prediction matrix. Validate with test_lineage_prediction (test_lineages), which computes TP, FP, precision, and sensitivity across confidence intervals; on the Biddy et al. MEF dataset at confidence cutoff 50 expect precision ~80% (±15%), sensitivity ~22% (±12%), and false positive rate FP/(FP+TN) under 0.07% (±0.08%). Optionally visualize and post-process predictions with visualize_as_network, prediction_to_lineage_information, and trim_network_to_size (e.g., max_size=2, cutoff=50).

## Related tools

- **GEMLI** (R package providing predict_lineages, test_lineage_prediction, visualize_as_network, and trim_network_to_size for lineage inference from scRNA-seq) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install GEMLI from GitHub via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0'))
- **R** (Runtime environment in which GEMLI is loaded and executed)
- **igraph** (Library required for network visualization of predicted lineages via visualize_as_network)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); GEMLI_items <- list(gene_expression = expr_matrix, barcodes = barcode_vector); predictions <- predict_lineages(GEMLI_items); test_lineage_prediction(GEMLI_items)
```

## Evaluation signals

- At confidence cutoff 50 on the MEF dataset (GSE99915), precision falls within 0.65–0.95 (reported mean 80% ± 15%)
- Sensitivity at cutoff 50 falls within 0.10–0.34 (reported mean 22% ± 12%)
- False positive rate computed as FP/(FP+TN) falls within 0.00–0.0015 (reported 0.07% ± 0.08%)
- test_lineage_prediction outputs a table containing TP, FP, precision, and sensitivity across confidence intervals that can be saved as gemli_mef_testing_results.csv
- predict_lineages completes repeated iterative clustering in roughly 2-3 minutes and returns a valid cell-vs-cell confidence-score matrix

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage tracing may require other approaches

## Evidence

- [abstract] GEMLI small lineage predictions reach ~80% precision, ~22% sensitivity at cutoff 50: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [methods] predict_lineages performs repeated iterative clustering: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] test_lineage_prediction validates predictions: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [readme] Installation via devtools install_github: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [discussion] Limitation on distantly related cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
