---
name: knn-ensemble-cell-classification
description: Use when you need to label individual cells in a normalized scRNA-seq
  matrix (genes as rownames, cells as colnames) as malignant or non-malignant, especially
  when you have no annotated training cohort of your own, since five pretrained k-NN
  (k = 5) classifiers are shipped.
when_to_use_negative:
- The goal is fine-grained annotation of non-malignant subpopulations (immune, stromal,
  epithelial cell types) — the algorithm is limited to annotating cancer vs non-cancer
  cells and is not designed for identifying non-cancer subpopulations.
- The input matrix is raw counts that have not been library-depth normalized — PreCanCell_data()
  expects normalized expression values (e.g., TPM, CPM).
- The input is bulk RNA-seq (e.g., TCGA/CCLE/GDSC expression profiles) — the k-NN
  classifiers are trained on single-cell transcriptomes and target per-cell annotation.
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: PreCanCell
  role: 'R package executing the whole ensemble: PreCanCell_data() selects matched
    DEG features and scales expression to [0,1]; PreCanCell_classifier() runs the
    five k-NN (k = 5) models and majority vote; Custom_preprocessing()/Custom_classifier()
    train user-specific models'
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: k-NN implementation used by PreCanCell_classifier() to classify each cell
    (k = 5) against each of the five training datasets
  repo: https://github.com/davpinto/fastknn
- name: Seurat
  role: NormalizeData() provides the library-depth normalization of the input matrix
    and FindMarkers() (Bonferroni-adjusted Wilcoxon rank-sum test, P < 0.05) identified
    the TMG/NMG features; Seurat version >= 4.3.0 required
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:2a2dbce0a38119265bfb090a016fa65d14e5e387d35c208cb5a3e83109e0fe5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - CHETAH
  - PreCanCell
  - R
  - SCINA
  - SciBet
  - Seurat
  - SingleR
  - doParallel
  - fastknn
  - foreach
  - ikarus
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# knn-ensemble-cell-classification

## Summary

Ensemble binary cell-state classification from single-cell transcriptomes (PreCanCell): a normalized expression matrix is restricted to the shared DEG set (73 TMGs + 186 NMGs) from five cancer training datasets, min-max scaled to [0,1], and each cell is labeled cancer/non_cancer by the majority vote of five k-NN (k = 5) classifiers.

## When to use

Use when you need to label individual cells in a normalized scRNA-seq matrix (genes as rownames, cells as colnames) as malignant or non-malignant, especially when you have no annotated training cohort of your own, since five pretrained k-NN (k = 5) classifiers are shipped.

## When NOT to use

- The goal is fine-grained annotation of non-malignant subpopulations (immune, stromal, epithelial cell types) — the algorithm is limited to annotating cancer vs non-cancer cells and is not designed for identifying non-cancer subpopulations.
- The input matrix is raw counts that have not been library-depth normalized — PreCanCell_data() expects normalized expression values (e.g., TPM, CPM).
- The input is bulk RNA-seq (e.g., TCGA/CCLE/GDSC expression profiles) — the k-NN classifiers are trained on single-cell transcriptomes and target per-cell annotation.

## Inputs

- Normalized single-cell expression matrix (any library-depth normalization, e.g. TPM/CPM; rownames = genes, colnames = cells)
- Optional: ground-truth malignant/non-malignant cell annotations (e.g., GSE157220 pan-cancer cell lines as malignant, GSE126030 healthy-tissue immune cells as non-malignant) for metric computation
- Optional: user-provided annotated training cohort (counts + annotation) for Custom_preprocessing()/Custom_classifier()

## Outputs

- testdata: [0,1]-scaled expression matrix restricted to the matched DEG features (73 TMGs + 186 NMGs)
- Per-cell results table with Sample, freq_cancer, freq_non_cancer and pred_label (cancer / non_cancer)
- Evaluation metrics (accuracy, sensitivity, specificity, balanced accuracy, AUROC) against ground truth, when labels are available

## How to apply

Normalize raw counts with any library-depth normalization (e.g., TPM or CPM, e.g., via Seurat's NormalizeData()) so the matrix has genes as rownames and cells as colnames. Call PreCanCell_data(), which restricts the matrix to the matched DEG features — 73 tumor marker genes (TMGs) and 186 normal marker genes (NMGs) originally identified with Seurat's FindMarkers() at Bonferroni-adjusted Wilcoxon rank-sum P < 0.05 as the intersection across the five training datasets (RCC, HNSCC, melanoma, LUAD, BC) — and min-max scales each gene's expression to [0,1]. Then call PreCanCell_classifier(testdata, cores = N): every cell is classified by k-NN (k = 5, via the fastknn package) against each of the five training sets, and the final cancer/non_cancer label is the majority vote of the five results, returned per cell with freq_cancer and freq_non_cancer proportions. If you do have your own annotated training cohort with counts and annotations, use Custom_preprocessing() and Custom_classifier() to build a user-trained model instead. The rationale is that voting across five cross-cancer k-NN models on conserved DEGs suppresses dataset-specific signal, yielding above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC in all 19 tested single-cell datasets.

## Related tools

- **PreCanCell** (R package executing the whole ensemble: PreCanCell_data() selects matched DEG features and scales expression to [0,1]; PreCanCell_classifier() runs the five k-NN (k = 5) models and majority vote; Custom_preprocessing()/Custom_classifier() train user-specific models) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (k-NN implementation used by PreCanCell_classifier() to classify each cell (k = 5) against each of the five training datasets) — https://github.com/davpinto/fastknn
- **Seurat** (NormalizeData() provides the library-depth normalization of the input matrix and FindMarkers() (Bonferroni-adjusted Wilcoxon rank-sum test, P < 0.05) identified the TMG/NMG features; Seurat version >= 4.3.0 required)

## Examples

```
testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- PreCanCell_data() output must contain only the matched DEG features (73 TMGs + 186 NMGs) with every gene's values min-max scaled into [0,1].
- Classifier results must give pred_label in {cancer, non_cancer} with freq_cancer + freq_non_cancer = 1 for each cell (e.g., 0.8/0.2 -> cancer).
- On the synthetic merged dataset (GSE157220 + GSE126030), metrics should reproduce approximately accuracy 0.995, sensitivity 0.998, specificity 0.993, balanced accuracy 0.995, AUROC 0.995, with FPR = 0.0089 on GSE126030 and FNR = 0.0041 on GSE157220.
- On new test sets, all five metrics (accuracy, sensitivity, specificity, balanced accuracy, AUROC) should stay above 0.8.
- In comparator benchmarks against CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR and ikarus, PreCanCell should rank first or second in accuracy.

## Limitations

- Reported training-set predictive performance may be overestimated because the predicted sets were involved in DEG feature selection.
- Only annotates cancer vs non-cancer; it does not resolve non-cancer cells into immune/stromal/epithelial subpopulations.
- No version number is stated for the PreCanCell R package; installation depends on Seurat >= 4.3.0 and the GitHub-only fastknn package.

## Evidence

- [readme] The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells.: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [readme] With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5).: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)."
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] running `PreCanCell_classifier()` function to predict malignant and non-malignant cells: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [methods] the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [discussion] the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection: "the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection"
- [discussion] this algorithm is limited to annotating cancer and non-cancer cells, while it is not designed for identifying subpopulations of non-cancer cells: "this algorithm is limited to annotating cancer and non-cancer cells, while it is not designed for identifying subpopulations of non-cancer cells"
