---
name: multinomial-logistic-timepoint-classification
description: Use when predicting the circadian time of a single transcriptomic sample
  (bulk or pseudobulk) after training on a paired-expression/timepoint time series,
  with rhythmic genes as platform-agnostic features.
when_to_use_negative:
- Predicting circadian time of individual cells — tauFisher cannot overcome the high
  sequencing dropout rate in scRNAseq and only works on bulk or pseudobulk data
- You need uncertainty quantification or confidence scores — tauFisher does not output
  them; consider Tempo for uncertainty estimates in single-cell data
- Projecting test data onto the timescale of diseased samples — a time series from
  diseased individuals is required, which a model trained on healthy data cannot provide
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: tauFisher
  role: 'R package providing the full pipeline: get_FDAcurves, create_DiffMatrix,
    scale_DiffMatrix, train_tauFisher, test_tauFisher, calc_error'
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: meta2d (JTK_Cycle and Lomb-Scargle) via run_metacycle to detect genes with
    24-hour periods and JTK p-values < 0.05
- name: fda
  role: Fourier-basis functional data analysis (numbasis=5) to smooth discrete time-expression
    curves into continuous functions
- name: stats
  role: prcomp (scale=FALSE) to compute the PCA embedding of the scaled pairwise difference
    features
- name: nnet
  role: multinom fits the multinomial logistic regression on PC1 and PC2 to predict
    releveled circadian time
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:ab3ad48221cefa5c269da94d5dc74c34cd905bc604cca29b914343a0ea6ff1a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# multinomial-logistic-timepoint-classification

## Summary

tauFisher's timepoint classifier: pairwise expression differences of rhythmic predictor genes are PCA-embedded and a multinomial logistic regression (nnet::multinom) on the first two principal components predicts the circadian time (CT) of a single transcriptomic sample, independent of assay platform.

## When to use

Use when predicting the circadian time of a single transcriptomic sample (bulk or pseudobulk) after training on a paired-expression/timepoint time series, with rhythmic genes as platform-agnostic features.

## When NOT to use

- Predicting circadian time of individual cells — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and only works on bulk or pseudobulk data
- You need uncertainty quantification or confidence scores — tauFisher does not output them; consider Tempo for uncertainty estimates in single-cell data
- Projecting test data onto the timescale of diseased samples — a time series from diseased individuals is required, which a model trained on healthy data cannot provide

## Inputs

- Training transcriptomic time series (e.g., GEO GSE38622 skin microarray) with sample-level adjusted circadian time labels
- MetaCycle meta2d output file of rhythmicity statistics (JTK p-values, periods)
- Periodic gene list (top ten 24-hour JTK p<0.05 genes plus 9 core clock genes)
- Single within-sample-normalized test transcriptomic sample with unknown circadian time

## Outputs

- Trained model RData artifact containing predictor genes, PCA embedding, and fitted multinomial logistic model
- Predicted circadian time (CT) for each test sample
- Error metrics from calc_error (e.g., % correct within 2 hours, RMSE)

## How to apply

First identify predictors by running MetaCycle meta2d (JTK_Cycle and Lomb-Scargle) on the training series via run_metacycle, then find_periodic_genes keeps the top ten genes with a 24-hour period and JTK_pvalue < 0.05, plus the 9 core clock genes (~18 predictors for GSE38622). Log2-transform expression as log2(x+1), smooth each gene's time course with functional data analysis using a Fourier basis with numbasis=5 (get_FDAcurves), wrapping times to 24 hours, then compute all pairwise gene differences (create_DiffMatrix) and rescale them to [0,1] (scale_DiffMatrix) — this within-sample normalization makes gene-pair differences depend only on circadian time, hence platform-independent. Run PCA with stats::prcomp (scale=FALSE) keeping 2 dimensions, relevel the CT factor to the minimum time point, and fit nnet::multinom(CT24_relevel ~ PC1 + PC2). Save the trained model list (predictor genes, PCA embedding, fitted model) as an RData artifact together with the predictor gene list and runnable training script; validate by scoring predictions within 2 hours of true circadian time via calc_error.

## Related tools

- **tauFisher** (R package providing the full pipeline: get_FDAcurves, create_DiffMatrix, scale_DiffMatrix, train_tauFisher, test_tauFisher, calc_error) — github.com/micnngo/tauFisher
- **MetaCycle** (meta2d (JTK_Cycle and Lomb-Scargle) via run_metacycle to detect genes with 24-hour periods and JTK p-values < 0.05)
- **fda** (Fourier-basis functional data analysis (numbasis=5) to smooth discrete time-expression curves into continuous functions)
- **stats** (prcomp (scale=FALSE) to compute the PCA embedding of the scaled pairwise difference features)
- **nnet** (multinom fits the multinomial logistic regression on PC1 and PC2 to predict releveled circadian time)

## Examples

```
trained_JTK <- train_tauFisher(periodic_genes=genes$JTK); result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes)
```

## Evaluation signals

- Artifact must contain all three model components (predictor genes, PCA embedding, fitted model); predictor gene list must match the expected ~18-gene set (top-ten 24h JTK p<0.05 genes plus 9 core clock genes)
- Predictions scored via calc_error are correct when within 2 hours of true circadian time; report accuracy and RMSE
- Scaling of difference features to [0,1] succeeded and PCA was run with scale=FALSE on the rescaled pairwise differences
- CT factor was releveled to the minimum time point before fitting nnet::multinom, and only the first two principal components (ndims=2) are used
- Model generalizes across assay platforms: train on one platform/dataset and predict on a held-out replicate or independent dataset

## Limitations

- Within-sample normalization rests on an unverified assumption that the differences between predictor genes are solely dependent on circadian time
- tauFisher's performance on human blood samples is subpar despite strong mouse benchmark results
- Ordinal regression alternatives were not as robust as the multinomial model and failed to run on the entire set of time points
- Choosing the top ten diurnal genes does not guarantee the best performance, though it is a safe and reasonable choice across datasets
- tauFisher cannot predict circadian time at a single-cell level due to high scRNAseq dropout, and outputs no confidence scores

## Evidence

- [discussion] PCA projection and multinomial regression: "linearly transforming the scaled differences using principal component analysis and fitting a multinomial logistic regression on the first two principal components"
- [discussion] predictor gene selection: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [discussion] top-ten selection: "It then selects the top ten statistically significant genes with a 24-hour period length."
- [discussion] FDA smoothing rationale: "tauFisher uses functional data analysis to represent the discrete time points as continuous functions"
- [discussion] normalization assumption: [evidence span withheld — anchor status: not_found]
- [readme] package requirements: [evidence span withheld — anchor status: not_found]
