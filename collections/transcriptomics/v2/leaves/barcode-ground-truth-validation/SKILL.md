---
name: barcode-ground-truth-validation
description: Use when lineage predictions from scRNA-seq gene expression alone need
  quantitative validation against a known barcode lineage annotation, or when selecting
  a confidence cutoff with acceptable precision (e.g., precision = 1.0 at confidence
  80).
when_to_use_negative:
- No independent barcode or lineage-tracing ground truth exists for the dataset, so
  no TP/FP can be computed
- Cell population contains only distantly related cells, where GEMLI predictions themselves
  are unreliable and cannot meaningfully be benchmarked
- Goal is long-term lineage tracing over weeks or months spanning many divisions,
  beyond GEMLI's ~1–5 cell division scope
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: GEMLI
  role: predict_lineages produces the cell-by-cell confidence matrix; test_lineage_prediction
    compares it to the barcode ground truth to yield TP/FP/precision/sensitivity across
    cutoffs
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: installs the GEMLI package from GitHub via install_github('UPSUTER/GEMLI',
    subdir='GEMLI_package_v0')
- name: topGO
  role: performs GO biological process over-representation analysis on memory gene
    sets derived from ground-truth vs predicted lineages for the correlation check
- name: xcalibr
  role: extracts and counts CRISPR barcode reads from short reads to build the barcode
    ground-truth reference
  repo: https://github.com/NKI-GCF/xcalibr
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:c4d58d418b5d05159d1abd97d1cffc2d4d33b252f7f93817fe6ca300750175f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - topGO
  - xcalibr
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# barcode-ground-truth-validation

## Summary

Validates GEMLI lineage predictions from scRNA-seq data against an independent CRISPR/CellTag barcode ground truth by computing TP, FP, precision, and sensitivity across lineage-confidence cutoffs, and additionally checks recovery of gene-expression-memory genes and GO terms (Fig. 3a).

## When to use

Use when lineage predictions from scRNA-seq gene expression alone need quantitative validation against a known barcode lineage annotation, or when selecting a confidence cutoff with acceptable precision (e.g., precision = 1.0 at confidence 80).

## When NOT to use

- No independent barcode or lineage-tracing ground truth exists for the dataset, so no TP/FP can be computed
- Cell population contains only distantly related cells, where GEMLI predictions themselves are unreliable and cannot meaningfully be benchmarked
- Goal is long-term lineage tracing over weeks or months spanning many divisions, beyond GEMLI's ~1–5 cell division scope

## Inputs

- Quality-controlled, normalized scRNA-seq gene expression matrix (rows = genes, columns = cells), e.g. from GSE173958 (GSM5283486, GSM5283491) or GSE99915 MEF data
- Ground-truth lineage barcode dictionary (values = barcode ID, names = cell IDs) from evolvable CRISPR barcoding or CellTag lineage tracing
- GEMLI-predicted lineage confidence matrix from predict_lineages

## Outputs

- testing_results table with TP, FP, precision and sensitivity per confidence level (0–100)
- Precision–sensitivity curve (e.g. precision_sensitivity_curve.png)
- Chosen confidence cutoff with verified precision (e.g. precision = 1.0 at confidence 80)
- Memory gene sets and topGO GO enrichment scores with Spearman correlation (e.g. correlation_results.csv)

## How to apply

Assemble a GEMLI_items list containing the quality-controlled, normalized gene expression matrix (rows = genes, columns = cells) and a ground-truth barcode dictionary (values = barcode ID, names = cell IDs). Run GEMLI::predict_lineages to obtain the all-cells-against-all-cells matrix of lineage confidence scores from repeated iterative clustering. Run GEMLI::test_lineage_prediction (test_lineages) against the barcode ground truth to obtain TP, FP, precision and sensitivity for confidence levels from 0 to 100; optionally set plot_results=T to render the precision–sensitivity curve. Choose a cutoff meeting the desired precision (e.g., in the mouse pancreatic cancer lung metastasis dataset, GEMLI reached 100% precision at confidence level 80 while assigning more cells to lineages than the barcoding ground truth). As a complementary validation, identify memory genes separately from barcode ground-truth lineages and predicted lineages, run GO biological process over-representation with topGO, and compute the Spearman rank correlation between enrichment scores — a correlation exceeding 0.7 indicates faithful recovery, consistent with Fig. 3a.

## Related tools

- **GEMLI** (predict_lineages produces the cell-by-cell confidence matrix; test_lineage_prediction compares it to the barcode ground truth to yield TP/FP/precision/sensitivity across cutoffs) — https://github.com/UPSUTER/GEMLI
- **devtools** (installs the GEMLI package from GitHub via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0'))
- **topGO** (performs GO biological process over-representation analysis on memory gene sets derived from ground-truth vs predicted lineages for the correlation check)
- **xcalibr** (extracts and counts CRISPR barcode reads from short reads to build the barcode ground-truth reference) — https://github.com/NKI-GCF/xcalibr

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); results <- test_lineage_prediction(GEMLI_items, plot_results=T)
```

## Evaluation signals

- testing_results table shows precision = 1.0 (100%) at the chosen confidence level (e.g., 80) on the mouse pancreatic cancer lung metastasis dataset
- Predicted lineage assignments cover at least as many cells as the CRISPR barcoding ground truth at that cutoff
- Precision–sensitivity curve renders across the full 0–100 confidence range without errors
- Spearman rank correlation between GO enrichment scores of ground-truth-lineage and predicted-lineage memory gene sets exceeds 0.7, consistent with Fig. 3a
- Barcode dictionary keys (cell IDs) match the column names of the gene expression matrix before testing

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, degrading benchmark precision
- Large variability in growth rates between lineages may decrease GEMLI's performance
- GEMLI's scope is limited to lineages over about 1–5 cell divisions; long-term tracing requires naturally occurring genetic marks over weeks or months

## Evidence

- [methods] Test against barcode ground truth to obtain TP, FP, precision and sensitivity: "The `test_lineage_prediction` function again takes our `GEMLI_items` as input"
- [abstract] GEMLI robustly identifies small to medium-sized lineages solely from scRNA-seq: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [readme] GEMLI installed from GitHub with devtools: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [methods] Lineage prediction relies on iterative clustering of expression-based confidence scores: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [readme] GSE99915 MEF dataset with CellTag barcodes is publicly available for ground-truth comparisons: [evidence span withheld — anchor status: not_found]
