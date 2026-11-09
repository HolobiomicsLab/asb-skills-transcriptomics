---
name: r-python-interoperable-tool-execution
description: Use when reproducing a multi-tool comparison where comparators are implemented
  in different languages (e.g., R cell-type annotators vs. a Python classifier like
  ikarus) and results must be computed on the same inputs and metrics.
when_to_use_negative:
- The task requires annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  rather than the binary cancer vs. non-cancer decision this comparison covers
- Only one tool in one language is needed; the R/Python orchestration overhead is
  unnecessary for a single-method run
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: PreCanCell
  role: R-package ensemble (DEG features, k-NN k=5, majority vote over five training
    sets) being benchmarked
  repo: https://github.com/WangX-Lab/PreCanCell
- name: CHETAH
  role: R-package comparator run with thresh = 0
- name: SciBet
  role: R-package comparator
- name: SCINA
  role: R-package comparator run with allow_unknown = 0
- name: scmap-cell
  role: R-package comparator
- name: scmap-cluster
  role: R-package comparator
- name: SingleR
  role: R-package comparator
- name: ikarus
  role: Python-package comparator run on the same test matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:4f3bde7d1f7d232c89b259a35b2cf706a14489920eedb1391de30fd1a5c2afa3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - CHETAH
  - PreCanCell
  - SCINA
  - SciBet
  - SingleR
  - ikarus
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# r-python-interoperable-tool-execution

## Summary

Runs a benchmark of cell-annotation algorithms split across two languages: CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, and SingleR via their R packages, ikarus via its Python package, and PreCanCell via its R package, so all eight are evaluated on identical test sets.

## When to use

Use when reproducing a multi-tool comparison where comparators are implemented in different languages (e.g., R cell-type annotators vs. a Python classifier like ikarus) and results must be computed on the same inputs and metrics.

## When NOT to use

- The task requires annotating non-cancer cell subpopulations (immune, stromal, epithelial) rather than the binary cancer vs. non-cancer decision this comparison covers
- Only one tool in one language is needed; the R/Python orchestration overhead is unnecessary for a single-method run

## Inputs

- Normalized scRNA-seq expression matrices (rownames = genes, colnames = cells) for 11 test datasets
- Malignant/non-malignant ground-truth annotations per cell for each test set
- Five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) for PreCanCell
- R packages: CHETAH, SciBet, SCINA, scmap, SingleR, PreCanCell (with Seurat >= 4.3.0 and fastknn)
- Python package: ikarus

## Outputs

- Per-algorithm, per-dataset classification labels (cancer / non_cancer)
- Comparison table of accuracy, sensitivity, specificity, balanced accuracy, and AUROC for all eight algorithms across the 11 test sets
- Accuracy-based ranking of the eight algorithms

## How to apply

Load the normalized scRNA-seq expression matrices for the 11 test datasets (e.g., P2_BCC, P3_HCC, P7_GBM) with malignant/non-malignant ground-truth labels. Run CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, and SingleR through their R packages with default parameters, overriding SCINA to 'allow_unknown = 0' and CHETAH to 'thresh = 0' so all tools force a two-class decision. Run ikarus through its Python package on the same matrices. Run PreCanCell in R using the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) and DEG features with k-NN (k = 5) ensemble classification via PreCanCell_data() then PreCanCell_classifier(). Compute accuracy, sensitivity, specificity, balanced accuracy, and AUROC per algorithm per test set, then aggregate into one comparison table ranked by accuracy. The rationale is that identical inputs, forced-label settings, and shared metrics make cross-language results directly comparable.

## Related tools

- **PreCanCell** (R-package ensemble (DEG features, k-NN k=5, majority vote over five training sets) being benchmarked) — https://github.com/WangX-Lab/PreCanCell
- **CHETAH** (R-package comparator run with thresh = 0)
- **SciBet** (R-package comparator)
- **SCINA** (R-package comparator run with allow_unknown = 0)
- **scmap-cell** (R-package comparator)
- **scmap-cluster** (R-package comparator)
- **SingleR** (R-package comparator)
- **ikarus** (Python-package comparator run on the same test matrices)

## Examples

```
devtools::install_github("davpinto/fastknn"); devtools::install_github("WangX-Lab/PreCanCell"); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- The aggregated table contains rows for all eight algorithms across all 11 test sets with accuracy, balanced accuracy, and AUROC columns populated
- PreCanCell ranks first or second in accuracy on each test set (highest on three, second-highest on three) as reported
- No 'unknown' labels appear for SCINA or CHETAH results, confirming allow_unknown = 0 and thresh = 0 were applied
- Each of the 19 evaluated single-cell datasets shows PreCanCell accuracy, sensitivity, specificity, balanced accuracy, and AUROC above 0.8
- Metrics are computed against the same ground-truth annotations for every algorithm, so per-dataset denominators match across tools

## Limitations

- Compared algorithms (CHETAH, SciBet, scmap-cell, scmap-cluster, SingleR, SCINA, ikarus) are named but no software versions are stated
- No accession IDs or repository links are provided for the named test datasets (e.g., P1_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM)
- PreCanCell's training-set performance may be overestimated because the predicted sets were involved in feature selection

## Evidence

- [abstract] seven other algorithms, including CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, and ikarus: "seven other algorithms, including CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, and ikarus"
- [results] PreCanCell ranked first or second in accuracy: [evidence span withheld — anchor status: not_found]
- [intro] k-NN (k=5) classification: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
- [methods] PreCanCell_classifier usage: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [readme] Install PreCanCell via devtools: "devtools::install_github("WangX-Lab/PreCanCell")"
