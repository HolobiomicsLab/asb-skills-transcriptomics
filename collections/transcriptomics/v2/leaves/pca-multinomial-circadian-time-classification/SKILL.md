---
name: pca-multinomial-circadian-time-classification
description: Use when you have bulk or pseudobulk transcriptomic data with known circadian
  timestamps for training and need to predict the circadian time of single, untimestamped
  samples across assay platforms.
when_to_use_negative:
- 'Input is single-cell level scRNAseq: tauFisher cannot overcome the high sequencing
  dropout rate and cannot predict circadian time at a single cell level'
- 'You need uncertainty quantification or confidence scores: tauFisher does not output
  them (unlike Tempo-style Bayesian approaches)'
- The training time series is incomplete/unavailable or from diseased individuals
  when you want diseased-sample timescale projection without diseased training data
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: Core pipeline providing train_tauFisher and test_tauFisher for the PCA + multinomial
    logistic regression circadian-time classifier
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Runs JTK_Cycle and Lomb-Scargle (run_metacycle) to identify rhythmic genes
    used as classifier predictors
- name: fda
  role: Functional data analysis with Fourier basis (K=5) to smooth discrete time
    points into continuous expression functions
- name: nnet
  role: Fits the multinomial logistic regression on the first two principal components
- name: stats
  role: PCA projection of scaled pairwise gene-expression differences
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:8449b83377cd6203082776ec8c2ca4e290a300e87b8b24643e49f03033c8494b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# pca-multinomial-circadian-time-classification

## Summary

Train tauFisher's supervised circadian-time classifier: log2-transform rhythmic predictor genes, scale pairwise gene-expression differences, smooth with Fourier-basis FDA (numbasis=5), project via PCA, and fit a multinomial logistic regression on the first two principal components to predict circadian time from a single transcriptomic sample.

## When to use

Use when you have bulk or pseudobulk transcriptomic data with known circadian timestamps for training and need to predict the circadian time of single, untimestamped samples across assay platforms.

## When NOT to use

- Input is single-cell level scRNAseq: tauFisher cannot overcome the high sequencing dropout rate and cannot predict circadian time at a single cell level
- You need uncertainty quantification or confidence scores: tauFisher does not output them (unlike Tempo-style Bayesian approaches)
- The training time series is incomplete/unavailable or from diseased individuals when you want diseased-sample timescale projection without diseased training data

## Inputs

- Rhythmic gene list from find_periodic_genes (e.g., genes$JTK)
- Training expression matrix with circadian timestamps (bulk RNAseq/microarray, e.g., GSE157077 SCN, GSE38622 skin, replicate-offset time points)
- Test expression matrix (bulk RNAseq or pseudobulk scRNAseq, e.g., GSE83855, GSE117295 SCN pseudobulk, GSE223109 dermis pseudobulk)
- numbasis=5 Fourier basis parameter; nrep replicate count (e.g., nrep=3)

## Outputs

- Trained tauFisher model with chosen predictor genes (e.g., 18 skin genes, 16 surviving in dermis pseudobulk)
- Predicted circadian time per test sample
- Accuracy and RMSE from calc_error (e.g., accuracy 0.75, RMSE 4.704 on GSE83855; accuracy 0.833, RMSE 1.936 on SCN pseudobulk)
- predictions.csv / tauFisher_pseudobulk_predictions.csv with true time, predicted time, and metrics

## How to apply

After run_metacycle (JTK + Lomb-Scargle) and find_periodic_genes select rhythmic genes (e.g., JTK p < 0.05 present in both training and test data), train_tauFisher log2(x+1)-transforms the expression matrix, computes and scales pairwise differences between predictor genes for within-sample normalization, smooths time-expression curves with functional data analysis using a Fourier basis (numbasis=5), runs PCA, and fits a multinomial logistic regression (nnet) on the first two principal components. At prediction, test_tauFisher subsets the test sample to the chosen predictor genes (e.g., 18 skin genes from GSE38622), computes the same scaled pairwise differences, projects onto the training PCA embedding, and outputs a predicted circadian time; predictions are scored as correct when within 2 hours of truth via calc_error. This within-sample normalization rests on pairwise gene differences depending only on circadian time, which is what makes the model platform-generalizable (e.g., microarray training to RNAseq or pseudobulk testing).

## Related tools

- **tauFisher** (Core pipeline providing train_tauFisher and test_tauFisher for the PCA + multinomial logistic regression circadian-time classifier) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Runs JTK_Cycle and Lomb-Scargle (run_metacycle) to identify rhythmic genes used as classifier predictors)
- **fda** (Functional data analysis with Fourier basis (K=5) to smooth discrete time points into continuous expression functions)
- **nnet** (Fits the multinomial logistic regression on the first two principal components)
- **stats** (PCA projection of scaled pairwise gene-expression differences)

## Examples

```
trained_JTK <- train_tauFisher(periodic_genes=genes$JTK, df=bulk_adj, timepoints=time_adj, numbasis=5, nrep=3); result_JTK <- test_tauFisher(chosen_genes=trained_JTK$Genes, df=test_df)
```

## Evaluation signals

- calc_error: prediction counted correct when within 2 hours of the labeled circadian time; benchmark accuracies 0.75 (GSE83855), 0.833 (SCN pseudobulk), 0.778 (dermis pseudobulk)
- RMSE in the 1.9-4.7 range on published benchmarks (1.936 SCN pseudobulk, 2.198/2.778 dermis, 4.704 cross-platform skin)
- Trained model retains the expected number of predictor genes present in test data (e.g., 18 skin genes, dropping to 16 after single-cell dropout in GSE223109)
- predictions file (predictions.csv / tauFisher_pseudobulk_predictions.csv) contains true time, predicted time, accuracy, and RMSE columns
- Training pipeline order verified: rhythmic-gene selection precedes train_tauFisher with log2(x+1) transform, scaled pairwise differences, FDA smoothing (numbasis=5), then PCA + multinomial regression

## Limitations

- Performance on human blood samples is subpar (mouse benchmarks are strong)
- Cannot predict circadian time at single-cell level due to scRNAseq dropout; pseudobulk aggregation is required
- No uncertainty quantification or confidence scores are output
- Within-sample normalization assumes pairwise predictor-gene expression differences depend solely on circadian time, an unverified assumption
- Choosing the top ten diurnal genes as predictors is safe but not guaranteed optimal; ordinal regression alternatives failed to run on the full set of time points

## Evidence

- [methods] Fourier-basis FDA smoothing, PCA, and multinomial logistic regression on the first two principal components: [evidence span withheld — anchor status: not_found]
- [abstract] tauFisher predicts circadian time via correlations between rhythmic genes: "accurately predicts circadian time from a single transcriptomic sample by finding correlations between rhythmic genes within the sample"
- [results] Cross-platform accuracy metric: [evidence span withheld — anchor status: not_found]
- [discussion] Single-cell level limitation: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [readme] README: package dependencies and installation: [evidence span withheld — anchor status: external_doc]
