---
name: gene-pair-difference-normalization
description: Use when you must timestamp individual bulk or scRNAseq pseudobulk transcriptomic
  samples (e.g., microarray or RNAseq) whose circadian phase is unknown and no matched
  control series exists for cross-sample normalization.
when_to_use_negative:
- 'Predicting circadian time at single-cell resolution: tauFisher cannot overcome
  the high sequencing dropout rate in scRNAseq and only works on pseudobulk-aggregated
  samples'
- 'Predicting timescales of diseased samples while trained on healthy data: a time
  series from diseased individuals is required for that projection'
- Datasets where gene-pair differences plausibly depend on factors other than circadian
  time (e.g., circadian-disturbed/diseased tissue), since the within-sample normalization
  rests on that unverified assumption
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tauFisher
  role: Provides create_DiffMatrix and scale_DiffMatrix to compute and rescale pairwise
    predictor-gene expression differences, plus train_tauFisher/test_tauFisher that
    wrap PCA and multinomial regression on those features
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: meta2d with JTK_Cycle and Lomb-Scargle identifies the rhythmic 24-hour genes
    that become the predictor set whose pairwise differences are computed
- name: stats
  role: stats::prcomp (scale=FALSE) performs PCA on the scaled difference features
- name: nnet
  role: nnet::multinom fits the multinomial logistic regression on the first two principal
    components of the scaled differences
- name: fda
  role: Functional data analysis with Fourier basis (numbasis=5) smooths gene time
    courses before pairwise differences are computed
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:8d8385683a80cbdaeebd058a4c07740e7f8ef7cb021a39279851eacff05d7029
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - stringr
  - tauFisher
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# gene-pair-difference-normalization

## Summary

tauFisher's within-sample normalization: circadian time is predicted from a single transcriptomic sample by computing and scaling expression differences for each pair of rhythmic predictor genes, then projecting the scaled differences via PCA into a multinomial logistic regression on the first two principal components. This makes the feature vector dependent only on circadian time, not on library size or assay platform.

## When to use

Use when you must timestamp individual bulk or scRNAseq pseudobulk transcriptomic samples (e.g., microarray or RNAseq) whose circadian phase is unknown and no matched control series exists for cross-sample normalization.

## When NOT to use

- Predicting circadian time at single-cell resolution: tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and only works on pseudobulk-aggregated samples
- Predicting timescales of diseased samples while trained on healthy data: a time series from diseased individuals is required for that projection
- Datasets where gene-pair differences plausibly depend on factors other than circadian time (e.g., circadian-disturbed/diseased tissue), since the within-sample normalization rests on that unverified assumption

## Inputs

- Bulk transcriptomic time-series training matrix with known circadian-time labels (e.g., GSE38622 microarray or GSE157077 RNAseq), gene IDs upper-cased, duplicates averaged, samples ordered by adjusted circadian time
- MetaCycle meta2d output (JTK_Cycle and Lomb-Scargle results) for rhythmic-gene selection
- Single unlabeled test transcriptomic sample (bulk or single-cell pseudobulk, e.g., GSE83855, GSE117295, GSE223109)

## Outputs

- Scaled pairwise gene-difference feature matrix (scale_DiffMatrix output in [0,1])
- Trained model artifact: predictor gene list (~18 genes), PCA embedding, and fitted multinom model (RData)
- Predicted circadian time per test sample; accuracy and RMSE via calc_error (correct if within 2 hours of true time), e.g., predictions.csv

## How to apply

First select rhythmic predictor genes: run MetaCycle meta2d with both JTK_Cycle and Lomb-Scargle on the training time series (e.g., GSE38622 mouse skin microarray), then call find_periodic_genes keeping JTK p-value < 0.05 and 24-hour period, typically the top ten significant 24-hour genes plus core clock genes (~18 predictors). Log2-transform the expression matrix as log2(x+1), smooth each gene's time course with functional data analysis using a Fourier basis (numbasis=5), and compute all pairwise gene-expression differences with create_DiffMatrix, rescaling them to [0,1] with scale_DiffMatrix. Run PCA on the scaled difference features with stats::prcomp (scale=FALSE) and fit nnet::multinom on the first two principal components (ndims=2), releveleing the circadian-time factor to the minimum time point. For a new single sample, compute the same pairwise differences among the predictor genes, apply the training scaling, project onto the training PCA embedding, and take the multinomial regression's predicted circadian time. The rationale is that gene-pair differences within a sample are assumed to depend solely on circadian time, giving platform-independent, within-sample normalization from one sample alone.

## Related tools

- **tauFisher** (Provides create_DiffMatrix and scale_DiffMatrix to compute and rescale pairwise predictor-gene expression differences, plus train_tauFisher/test_tauFisher that wrap PCA and multinomial regression on those features) — github.com/micnngo/tauFisher
- **MetaCycle** (meta2d with JTK_Cycle and Lomb-Scargle identifies the rhythmic 24-hour genes that become the predictor set whose pairwise differences are computed)
- **stats** (stats::prcomp (scale=FALSE) performs PCA on the scaled difference features)
- **nnet** (nnet::multinom fits the multinomial logistic regression on the first two principal components of the scaled differences)
- **fda** (Functional data analysis with Fourier basis (numbasis=5) smooths gene time courses before pairwise differences are computed)

## Examples

```
trained_JTK <- train_tauFisher(periodic_genes=genes$JTK, df=bulk_adj, timepoints=time_adj, numbasis=5); result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, df=test_df)
```

## Evaluation signals

- Predictions are scored correct when within 2 hours of true circadian time via calc_error; benchmark expectations include accuracy 0.75 / RMSE 4.704 (GSE83855 skin) and accuracy 0.833 / RMSE 1.936 (GSE117295 SCN pseudobulk)
- The trained model artifact must contain all three components: predictor genes, PCA embedding, and fitted multinom model
- Predictor gene list matches the expected set (18 genes for GSE38622; 16 surviving after single-cell dropout for dermis pseudobulk)
- Scaled difference features must lie in [0,1] after scale_DiffMatrix
- Test-sample differences must be projected onto the training PCA embedding (not recomputed) before multinomial prediction

## Limitations

- The within-sample normalization assumes differences between predictor genes depend solely on circadian time, which is unverified
- Cannot predict circadian time at single-cell level due to scRNAseq dropout; predictor genes can be lost (e.g., A630005I04Rik and Ivl dropped in GSE223109 dermis)
- Performance on human blood samples is subpar compared to mouse benchmarks
- No uncertainty quantification or confidence scores are output; an imputation step for predictor gene expression is future work

## Evidence

- [discussion] pairwise differences are the core feature: "calculating and scaling the difference in expression for each pair of predictor genes"
- [discussion] PCA + multinomial on scaled differences: "linearly transforming the scaled differences using principal component analysis and fitting a multinomial logistic regression on the first two principal components"
- [discussion] normalization assumption: "One assumption of tauFisher’s within-sample normalization step is that the differences between predictor genes are solely dependent on circadian time"
- [discussion] predictor selection from rhythmic genes: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [methods] package invocation entry point: "trained_JTK <- train_tauFisher(periodic_genes=genes$JTK,"
