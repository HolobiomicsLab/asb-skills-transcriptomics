---
name: scrna-seq-cell-type-classification-benchmarking
description: Use when you need to compare malignant-vs-non-malignant cell prediction
  performance of an ensemble k-NN classifier against established cell-annotation tools
  on the same single-cell test datasets.
when_to_use_negative:
- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — PreCanCell is limited to annotating cancer vs non-cancer cells
- The input is raw unnormalized counts — PreCanCell_data() expects an already library-depth-normalized
  expression matrix (e.g., TPM or CPM)
- You need reproducible version-pinned comparisons without accession IDs — the article
  provides no versions for PreCanCell or the seven comparators and no accession IDs
  for several named test datasets
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: PreCanCell
  role: 'Ensemble classifier under benchmark: k-NN (k=5) with DEG features, five training
    sets, majority vote; R package'
  repo: https://github.com/WangX-Lab/PreCanCell
- name: CHETAH
  role: Comparator run with default parameters except thresh = 0
- name: SciBet
  role: Comparator cell-type classifier run with default parameters
- name: SCINA
  role: Comparator run with default parameters except allow_unknown = 0
- name: scmap-cell
  role: Comparator cell-annotation method run with default parameters
- name: scmap-cluster
  role: Comparator cell-annotation method run with default parameters
- name: SingleR
  role: Comparator cell-annotation method run with default parameters
- name: ikarus
  role: Comparator malignant/non-malignant classifier run via its Python package
- name: Seurat
  role: R package (version >= 4.3.0) whose NormalizeData() and FindMarkers() functions
    support preprocessing and DEG identification for the PreCanCell pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:1b57b92c4a40131eae9e0602fce54419dccf2ca993622d074644bd8707d20f48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - CHETAH
  - PreCanCell
  - SCINA
  - SciBet
  - Seurat
  - SingleR
  - ikarus
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scrna-seq-cell-type-classification-benchmarking

## Summary

Benchmark PreCanCell's malignant/non-malignant cell classification against CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, and ikarus on shared scRNA-seq test sets using accuracy, sensitivity, specificity, balanced accuracy, and AUROC.

## When to use

Use when you need to compare malignant-vs-non-malignant cell prediction performance of an ensemble k-NN classifier against established cell-annotation tools on the same single-cell test datasets.

## When NOT to use

- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — PreCanCell is limited to annotating cancer vs non-cancer cells
- The input is raw unnormalized counts — PreCanCell_data() expects an already library-depth-normalized expression matrix (e.g., TPM or CPM)
- You need reproducible version-pinned comparisons without accession IDs — the article provides no versions for PreCanCell or the seven comparators and no accession IDs for several named test datasets

## Inputs

- Normalized scRNA-seq expression matrices (rows = genes, columns = cells; any library-depth normalization e.g. TPM or CPM) for 11 test datasets (e.g., P2_BCC, P3_HCC, P7_GBM)
- Malignant/non-malignant ground-truth cell annotations for each test set
- Five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) with malignant/non-malignant labels
- DEG feature set (73 tumor marker genes and 186 normal marker genes)

## Outputs

- Per-algorithm, per-dataset metrics: accuracy, sensitivity, specificity, balanced accuracy, AUROC
- Aggregated comparison table across all 11 test sets for all eight algorithms, ranked by accuracy

## How to apply

Load normalized scRNA-seq expression matrices for the 11 test datasets (e.g., P2_BCC, P3_HCC, P7_GBM) with malignant/non-malignant ground-truth annotations. Run CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, and SingleR via their R packages and ikarus via its Python package with default parameters, except 'allow_unknown = 0' in SCINA and 'thresh = 0' in CHETAH. Run PreCanCell on the same test sets using the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) with DEGs (73 TMGs and 186 NMGs) as features and k-NN (k=5) ensemble classification by majority vote, after preprocessing with PreCanCell_data() which selects matched genes and applies [0,1]-scaling. Compute accuracy, sensitivity, specificity, balanced accuracy, and AUROC against ground-truth labels for each algorithm, aggregate into a comparison table across the 11 test sets, and rank algorithms by accuracy.

## Related tools

- **PreCanCell** (Ensemble classifier under benchmark: k-NN (k=5) with DEG features, five training sets, majority vote; R package) — https://github.com/WangX-Lab/PreCanCell
- **CHETAH** (Comparator run with default parameters except thresh = 0)
- **SciBet** (Comparator cell-type classifier run with default parameters)
- **SCINA** (Comparator run with default parameters except allow_unknown = 0)
- **scmap-cell** (Comparator cell-annotation method run with default parameters)
- **scmap-cluster** (Comparator cell-annotation method run with default parameters)
- **SingleR** (Comparator cell-annotation method run with default parameters)
- **ikarus** (Comparator malignant/non-malignant classifier run via its Python package)
- **Seurat** (R package (version >= 4.3.0) whose NormalizeData() and FindMarkers() functions support preprocessing and DEG identification for the PreCanCell pipeline)

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Comparison table contains rows for all eight algorithms across all 11 test sets with AUROC, accuracy, and balanced accuracy columns populated
- PreCanCell ranks first or second in accuracy on the 11 test sets (highest in three, second highest in three)
- PreCanCell achieves above 0.8 accuracy, sensitivity, specificity, balanced accuracy, and AUROC in all tested single-cell datasets
- PreCanCell_classifier() output contains per-cell columns Sample, freq_cancer, freq_non_cancer, and pred_label with values 'cancer' or 'non_cancer'
- Comparator runs use the specified non-default settings: allow_unknown = 0 in SCINA and thresh = 0 in CHETAH

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- No version numbers are stated for the PreCanCell R package or for the seven compared algorithms (CHETAH, SciBet, scmap-cell, scmap-cluster, SingleR, SCINA, ikarus)
- No accession IDs or repository links are provided for several named train/test datasets (P1_BC, T5_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM)

## Evidence

- [results] Ranked first or second in accuracy: [evidence span withheld — anchor status: not_found]
- [full_text] Comparison settings for SCINA and CHETAH: [evidence span withheld — anchor status: not_found]
- [abstract] Performance above 0.8: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [abstract] k-NN ensemble design: "a single cell is classified as malignant or non-malignant by k-NN (k = 5)"
- [readme] PreCanCell_data() input expectations: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [readme] PreCanCell_data() scaling behavior: [evidence span withheld — anchor status: external_doc]
- [abstract] Comparator tools named: "seven other algorithms, including CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, and ikarus"
