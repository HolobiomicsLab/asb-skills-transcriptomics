---
name: limma-moderated-gene-ranking
description: Use when ranking genes of an RNA-seq expression matrix with subtype labels
  to define a compact, DEG-based feature space (e.g. limma50) for regularized multiclass
  logistic regression subtyping.
when_to_use_negative:
- The goal is a single-sample caller applied to new profiles with a predefined reference
  (use the AWCA_PAM50 or released mLR models from the repo instead of re-running feature
  selection)
- Input is already a reduced signature/gene panel (e.g. the PAM50 gene set), so ranking
  the whole transcriptome to build a feature space is unnecessary
- Data are microarray profiles rather than RNA-seq (the article's mLR feature-selection
  pipeline targets global RNA-seq expression data)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: limma
  role: Computes the moderated differential-expression statistics used to rank genes
    and define the limmaN feature spaces for mLR training
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: multiclass logistic regression (mLR)
  role: Downstream learner whose embedded Lasso/L1 and Ridge/L2 regularizers consume
    the limmaN gene spaces for intrinsic subtyping
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:b80a71f20da6fc392bfe44148e492d2d6410ed867105050ed105862e37bf5e9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - limma
  - multiclass logistic regression (mLR)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# limma-moderated-gene-ranking

## Summary

Ranks genes from a TCGA RNA-seq breast cancer expression matrix using limma moderated (eBayes) statistics, in parallel with filter scores (Fisher, Mutual Information, Chi-squared, Spearman), to build limmaN gene feature spaces (N=10..1000) for training regularized multiclass logistic regression (mLR) subtyping models.

## When to use

Use when ranking genes of an RNA-seq expression matrix with subtype labels to define a compact, DEG-based feature space (e.g. limma50) for regularized multiclass logistic regression subtyping.

## When NOT to use

- The goal is a single-sample caller applied to new profiles with a predefined reference (use the AWCA_PAM50 or released mLR models from the repo instead of re-running feature selection)
- Input is already a reduced signature/gene panel (e.g. the PAM50 gene set), so ranking the whole transcriptome to build a feature space is unnecessary
- Data are microarray profiles rather than RNA-seq (the article's mLR feature-selection pipeline targets global RNA-seq expression data)

## Inputs

- TCGA RNA-seq gene expression matrix (genes x samples) with PAM50 intrinsic subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- Train/test split assignment per sample
- Feature-space sizes N (e.g. 10..1000, with N=50 as the best-performing setting)

## Outputs

- limma-moderated ranked gene list (per-gene moderated statistics)
- limmaN gene feature spaces (N=10..1000) for mLR training
- CV and held-out TCGA test accuracy and per-class (subtype) recalls per feature space, aggregated in accuracy_metrics.csv

## How to apply

Load the TCGA RNA-seq expression matrix with PAM50 subtype labels and fit a linear model per gene with limma, then apply the moderated eBayes statistics to obtain a ranked gene list. Cut the ranking at N=10..1000 to define nested limmaN gene spaces, alongside top-1000 spaces from Fisher, Mutual Information, Chi-squared, and Spearman filter scores. Train Lasso/L1 and Ridge/L2 regularized multiclass logistic regression (mLR) on each space, estimate generalization by cross-validation on the training split, then evaluate on the held-out TCGA test split. Compare CV accuracies against the reported 89-95% range and test accuracies against 86-88%; note that limma-derived signatures brought less convincing test performances than the best filter space (the limma50 DEG-based signature of 277 genes reached 88% test accuracy with macro-averaged recall 0.94, while Chi-squared's space achieved 93% Her2-Enriched recall). Write the aggregated accuracy and per-class recall table to accuracy_metrics.csv. A backward elimination variant of the limma ranking yields the limma50_BWE signature.

## Related tools

- **limma** (Computes the moderated differential-expression statistics used to rank genes and define the limmaN feature spaces for mLR training) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **multiclass logistic regression (mLR)** (Downstream learner whose embedded Lasso/L1 and Ridge/L2 regularizers consume the limmaN gene spaces for intrinsic subtyping) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
Rscript -e 'library(limma); design <- model.matrix(~0+subtype, data=pam50); fit <- eBayes(lmFit(expr, design)); ranked <- topTable(fit, number=1000)'
```

## Evaluation signals

- Cross-validation accuracies on the training split fall in the reported 89-95% range per feature space
- Held-out TCGA test accuracies fall in the 86-88% range; the limma50 space (277 genes) reproduces 88% test accuracy with macro-averaged recall 0.94
- Chi-squared filter space reproduces 93% Her2-Enriched recall, providing a benchmark for per-class recalls
- limmaN spaces are properly nested subsets of the moderated-statistics ranking (N=10..1000), and limma50 contains 277 genes
- Aggregated per-space accuracy and per-class recall table is written to accuracy_metrics.csv with one row per gene space

## Limitations

- limma-derived signatures brought less convincing performances on testing than the best filter-based spaces, despite strong CV results
- Performances are biased because published subtype labels were themselves obtained with the PAM50 assay, favoring spaces using PAM50 genes
- The Normal-like class, whose clinical significance remains undetermined, degrades performance, and few training samples with a vast number of features limit recognition of hard cases

## Evidence

- [abstract] regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome"
- [discussion] particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [discussion] limma-derived signatures brought less convincing performances on testing: "limma-derived signatures brought less convincing performances on testing"
- [discussion] limma50_BWE (this latter one from a further backward elimination strategy): "limma50_BWE (this latter one from a further backward elimination strategy)"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method or our regularized multiclass Logistic Regression models: [evidence span withheld — anchor status: not_found]
