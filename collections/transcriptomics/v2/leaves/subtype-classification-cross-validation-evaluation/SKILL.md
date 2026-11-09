---
name: subtype-classification-cross-validation-evaluation
description: Use when a supervised RNA-seq subtype classifier (mLR with Lasso/L1 or
  Ridge/L2) has been fitted on a feature-selected gene space and you need CV accuracy,
  held-out test accuracy, and per-class recall/precision tables to compare feature
  spaces.
when_to_use_negative:
- When no held-out test split or cross-validation scheme is available and only a single
  reference-based PAM50 call per sample exists — use the AWCA_PAM50_BC_subtyping code
  instead
- When the goal is unsupervised centroid-correlation subtyping rather than evaluation
  of a trained supervised mLR classifier
- When training labels were themselves produced by the PAM50 assay on the same gene
  panel — accuracy estimates are biased and the paper explicitly flags such performances
  as biased
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Source of limmaN differential-expression gene signatures (e.g., limma50, 277
    genes, best test accuracy of 88%) whose classifier performance this evaluation
    skill benchmarks; the paper notes limma-derived signatures brought less convincing
    performances on testing
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:c31671dc54342f21871c7e39dc19c68fda31b84d87aef32c80bca97ae04a6cdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - limma
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# subtype-classification-cross-validation-evaluation

## Summary

Estimates and verifies the generalization performance of regularized multiclass logistic regression (mLR) breast cancer intrinsic-subtype classifiers by combining cross-validation on a TCGA training split with held-out test evaluation, recording overall accuracy and per-subtype recalls. It operationalizes the paper's finding that ad-hoc gene selection on the global transcriptome improves mLR subtyping beyond the whole-transcriptome feature space.

## When to use

Use when a supervised RNA-seq subtype classifier (mLR with Lasso/L1 or Ridge/L2) has been fitted on a feature-selected gene space and you need CV accuracy, held-out test accuracy, and per-class recall/precision tables to compare feature spaces.

## When NOT to use

- When no held-out test split or cross-validation scheme is available and only a single reference-based PAM50 call per sample exists — use the AWCA_PAM50_BC_subtyping code instead
- When the goal is unsupervised centroid-correlation subtyping rather than evaluation of a trained supervised mLR classifier
- When training labels were themselves produced by the PAM50 assay on the same gene panel — accuracy estimates are biased and the paper explicitly flags such performances as biased

## Inputs

- Fitted regularized multiclass logistic regression (L1/L2) model per gene space (scikit-learn)
- TCGA RNA-seq expression matrix with PAM50 subtype labels, split into training and held-out test sets
- Candidate feature spaces: whole transcriptome, top-1000 filter gene spaces (Fisher, Mutual Information, Chi-squared, Spearman), limmaN DEG spaces (N=10..1000)

## Outputs

- Cross-validation accuracy per gene space (expected range 89-95%)
- Held-out TCGA test accuracy per gene space (expected range 86-88%)
- Per-subtype recall and precision table for the five intrinsic subtypes
- Aggregated accuracy and per-class recall table written to accuracy_metrics.csv

## How to apply

After fitting L1- or L2-regularized multiclass logistic regression (scikit-learn) on each candidate gene space (whole transcriptome, top-1000 filter spaces from Fisher, Mutual Information, Chi-squared, and Spearman scores, and limmaN DEG spaces for N=10..1000), estimate generalization by cross-validation on the training split and record per-space accuracy. Then evaluate each fitted model on the held-out TCGA test split, computing overall accuracy and per-class (subtype) recalls for the five classes Luminal A, Luminal B, Basal, Her2-enriched, and Normal-like. Benchmark computed CV accuracies against the paper's reported 89-95% range and test accuracies against 86-88%; specifically check the Chi-squared filter's Her2-enriched recall of 93% and limma50's 88% test accuracy. Aggregate the accuracy and per-class recall values into a single comparison table (accuracy_metrics.csv) to rank feature spaces. Note that limma-derived signatures may be less convincing on testing than on CV, so both metrics are required before concluding a feature space is superior.

## Related tools

- **limma** (Source of limmaN differential-expression gene signatures (e.g., limma50, 277 genes, best test accuracy of 88%) whose classifier performance this evaluation skill benchmarks; the paper notes limma-derived signatures brought less convincing performances on testing) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
from sklearn.linear_model import LogisticRegression; m = LogisticRegression(penalty='l1', multi_class='multinomial', solver='saga').fit(X_train, y_train); acc = m.score(X_test, y_test)
```

## Evaluation signals

- CV accuracies across the five tested feature spaces fall within the reported 89-95% range
- Held-out test accuracies fall within the reported 86-88% range
- Chi-squared-filter gene space yields a Her2-enriched recall of approximately 93%
- limma50 DEG signature (277 genes) reproduces 88% test accuracy with macro-averaged recall of 0.94 and macro-averaged precision of 0.70, the best among tested feature spaces
- accuracy_metrics.csv contains one row per gene space with overall accuracy and per-subtype recall columns for all five classes

## Limitations

- Published subtype labels were themselves obtained with the PAM50 assay on the same gene panel, so higher accuracies with mLRs using PAM50 genes or AWCA-PAM50 are biased by the nature of the published calls
- The Normal-like class, whose clinical significance remains undetermined, causes degradation of evaluation performance
- The few training samples and the vast number of features limit subtyping capability, mainly for the hardest-to-recognize cases; limma-derived signatures brought less convincing performances on testing despite strong CV results

## Evidence

- [abstract] regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome.: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome."
- [discussion] feature selection combined with embedded Lasso and Ridge regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [discussion] the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method ... limma-derived signatures brought less convincing performances on testing: [evidence span withheld — anchor status: not_found]
- [abstract] mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample: "mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method ... or our regularized multiclass Logistic Regression models: [evidence span withheld — anchor status: not_found]
