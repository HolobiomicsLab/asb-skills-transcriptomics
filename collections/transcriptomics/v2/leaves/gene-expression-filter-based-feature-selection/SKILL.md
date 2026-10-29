---
name: gene-expression-filter-based-feature-selection
description: Use when training mLR subtype classifiers on whole-transcriptome RNA-seq
  matrices where a reduced, informative gene space is needed and filter scores or
  limma DE statistics can define the feature set.
when_to_use_negative:
- The feature space is already reduced (e.g. the PAM50 gene panel or a predefined
  signature like limma50_BWE) and no further gene selection is required
- Labels are not intrinsic subtypes or the data are microarray profiles (e.g. Affymetrix
  GSE4922/GSE1456), where limma-derived signatures brought less convincing testing
  performance
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: limma
  role: Computes moderated differential-expression statistics used to rank genes and
    define the limmaN feature spaces
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: scikit-learn
  role: Trains the Lasso/L1 and Ridge/L2 regularized multiclass logistic regression
    on each selected gene feature space
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:0f6bf602cf084821c53f39074c8809f436c51cd119f6e209492965bc4607fc9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - limma
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-expression-filter-based-feature-selection

## Summary

Ranks genes in a TCGA breast cancer RNA-seq expression matrix using univariate filter scores (Fisher, Mutual Information, Chi-squared, Spearman) and limma moderated statistics, then defines top-N gene feature spaces (limmaN, N=10..1000) to improve regularized multiclass logistic regression (mLR) intrinsic subtyping accuracy.

## When to use

Use when training mLR subtype classifiers on whole-transcriptome RNA-seq matrices where a reduced, informative gene space is needed and filter scores or limma DE statistics can define the feature set.

## When NOT to use

- The feature space is already reduced (e.g. the PAM50 gene panel or a predefined signature like limma50_BWE) and no further gene selection is required
- Labels are not intrinsic subtypes or the data are microarray profiles (e.g. Affymetrix GSE4922/GSE1456), where limma-derived signatures brought less convincing testing performance

## Inputs

- TCGA RNA-seq gene expression matrix with PAM50 intrinsic subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- Train/test split of the labeled TCGA cohort

## Outputs

- Top-1000 filter-based gene spaces for Fisher, Mutual Information, Chi-squared, and Spearman
- limmaN gene signatures for N=10..1000 (e.g. limma50: 277 genes)
- Per-space cross-validation and held-out test accuracy and per-class (subtype) recall table written to accuracy_metrics.csv

## How to apply

Load the TCGA RNA-seq expression matrix with PAM50 subtype labels and compute Fisher, Mutual Information, Chi-squared, and Spearman filter scores plus limma moderated statistics for every gene. Build five candidate feature spaces: top-1000 genes per filter method and limmaN signatures for N=10..1000. Train scikit-learn regularized multiclass logistic regression (Lasso/L1 and Ridge/L2 embedded regularizers) on each gene space, estimate generalization by cross-validation on the training split, and evaluate on the held-out TCGA test split recording overall accuracy and per-class recalls. Expect the limma50 DEG-based signature (277 genes, N=50) to be optimal: 88% test accuracy with macro-averaged recall 0.94 and precision 0.70, and check the Chi-squared filter's Her2-Enriched recall of 93% against the reported 86-88% test accuracy range.

## Related tools

- **limma** (Computes moderated differential-expression statistics used to rank genes and define the limmaN feature spaces) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **scikit-learn** (Trains the Lasso/L1 and Ridge/L2 regularized multiclass logistic regression on each selected gene feature space)

## Examples

```
python train_mlr.py --expression tcga_rnaseq.csv --labels pam50_subtypes.csv --feature-space limma50 --estimator lasso --cv 5 --test-split heldout --out accuracy_metrics.csv
```

## Evaluation signals

- Cross-validation accuracies on the training set fall in the reported 89-95% range across feature spaces
- Held-out TCGA test accuracies fall in the 86-88% range
- Chi-squared filter space yields Her2-Enriched recall of 93%
- limma50 signature yields 88% test accuracy with macro-averaged recall 0.94 and precision 0.70, the best among tested spaces
- accuracy_metrics.csv is produced with per-space accuracy and per-class recall rows for all five feature spaces

## Limitations

- Performance is biased because published PAM50 subtype labels were themselves obtained with the PAM50 assay on the same gene panel, favoring PAM50-gene-based feature spaces
- The Normal-like class, whose clinical significance remains undetermined, degrades classification performance
- Few training samples and a vast number of features limit subtyping of hard-to-recognize cases
- limma-derived signatures brought less convincing performances on external testing than on training

## Evidence

- [abstract] regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome"
- [discussion] feature selection combined with embedded regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [discussion] limma signatures on testing: "limma-derived signatures brought less convincing performances on testing"
- [discussion] labels biased by PAM50 assay: "the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method are biased by the nature of the published subtype calls, obtained in their turn from the PAM50 assay"
- [readme] README usage of mLR models: "our regularized multiclass Logistic Regression models ([mLR intrinsic subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/mLR_BC_subtyping)) to assign breast cancer (BC) samples to"
