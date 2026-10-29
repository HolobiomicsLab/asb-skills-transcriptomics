---
name: leave-one-out-cross-validation
description: Use when you must rigorously quantify tauFisher prediction accuracy on
  a small labeled time-series (e.g., GSE83855 time-restricted feeding schedules ZT5-ZT9
  and ZT0-ZT4) without a separate test set.
when_to_use_negative:
- When a large independent test set with known circadian time labels is available
  — a train/test split or benchmark against ZeitZeiger/TimeSignatR is preferable to
  costly per-fold retraining
- When samples lack circadian time labels or do not span the 24-hour cycle, since
  training the multinomial model on PC1/PC2 per fold requires labeled time points
- When predicting at the single-cell level, since tauFisher cannot overcome the high
  sequencing dropout rate in scRNAseq
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: Core predictor retrained per fold via train_tauFisher and applied to the held-out
    sample via test_tauFisher
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: meta2d run (JTK_Cycle and Lomb-Scargle) via run_metacycle to identify rhythmic
    predictor genes before CV
- name: fda
  role: Fourier basis (numbasis=5) functional data analysis to smooth time-expression
    curves during per-fold training
- name: nnet
  role: multinom multinomial logistic regression on PC1 and PC2 within each training
    fold
- name: stats
  role: stats::prcomp PCA of the scaled pairwise gene-difference matrix per fold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:ed17703e385dae4b8972d3f664ea5837f119a03129787090dff6d2c5cb44c28c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# leave-one-out-cross-validation

## Summary

Evaluates tauFisher circadian-time prediction by repeatedly holding out one sample, retraining on the remaining samples, and predicting the held-out sample's circadian time, then scoring accuracy (within 2 hours of true time) and RMSE per feeding schedule.

## When to use

Use when you must rigorously quantify tauFisher prediction accuracy on a small labeled time-series (e.g., GSE83855 time-restricted feeding schedules ZT5-ZT9 and ZT0-ZT4) without a separate test set.

## When NOT to use

- When a large independent test set with known circadian time labels is available — a train/test split or benchmark against ZeitZeiger/TimeSignatR is preferable to costly per-fold retraining
- When samples lack circadian time labels or do not span the 24-hour cycle, since training the multinomial model on PC1/PC2 per fold requires labeled time points
- When predicting at the single-cell level, since tauFisher cannot overcome the high sequencing dropout rate in scRNAseq

## Inputs

- GSE83855 expression matrices (genes as rows, samples as columns) with circadian time (ZT) labels
- Feeding-schedule group labels for the ZT5-ZT9 and ZT0-ZT4 time-restricted feeding conditions

## Outputs

- Per-schedule LOOCV accuracy (fraction of predictions within 2 hours of true time)
- Per-schedule RMSE of circadian time prediction
- gse83855_loocv_metrics.csv with accuracy and RMSE for both feeding schedules

## How to apply

Load the expression matrix (genes as rows, samples as columns) with circadian time labels — here GSE83855 under time-restricted feeding schedules ZT5-ZT9 and ZT0-ZT4. For each schedule, identify rhythmic predictor genes with MetaCycle's meta2d (via run_metacycle using JTK_Cycle and Lomb-Scargle) and select the top ten JTK-significant 24-hour period genes (JTK_pvalue < 0.05) with find_periodic_genes. For every held-out sample, retrain tauFisher on the remaining samples with train_tauFisher (log2-transform expression, smooth time-expression curves with fda using a numbasis=5 Fourier basis, compute and scale the pairwise gene-difference matrix, run stats::prcomp PCA, fit nnet::multinom on PC1 and PC2), then predict the held-out circadian time with test_tauFisher. Score each prediction with calc_error, counting predictions within 2 hours of the true time as correct, and compute RMSE per schedule; assemble metrics into gse83855_loocv_metrics.csv. Correct application reproduces the reported values: accuracy 0.875 / RMSE 2.236 for ZT5-ZT9 and accuracy 1.0 / RMSE 1.061 for ZT0-ZT4, demonstrating robust gene correlations persist in disturbed systems.

## Related tools

- **tauFisher** (Core predictor retrained per fold via train_tauFisher and applied to the held-out sample via test_tauFisher) — github.com/micnngo/tauFisher
- **MetaCycle** (meta2d run (JTK_Cycle and Lomb-Scargle) via run_metacycle to identify rhythmic predictor genes before CV)
- **fda** (Fourier basis (numbasis=5) functional data analysis to smooth time-expression curves during per-fold training)
- **nnet** (multinom multinomial logistic regression on PC1 and PC2 within each training fold)
- **stats** (stats::prcomp PCA of the scaled pairwise gene-difference matrix per fold)

## Examples

```
trained <- train_tauFisher(periodic_genes=genes$JTK, data=train_mat, time=train_times); result <- test_tauFisher(chosen_genes=trained$Genes, data=test_mat); calc_error(result)
```

## Evaluation signals

- Accuracy for feeding ZT5-ZT9 must equal 0.875 and RMSE 2.236
- Accuracy for feeding ZT0-ZT4 must equal 1.0 and RMSE 1.061
- Each of the N samples is held out exactly once and predictions are scored with calc_error, counting correct only if within 2 hours of the true time
- Selected predictor genes are exactly the top ten JTK-significant genes with 24-hour period and JTK_pvalue < 0.05 per schedule
- Final metrics table gse83855_loocv_metrics.csv contains one accuracy and one RMSE row per feeding schedule

## Limitations

- tauFisher's performance on human blood samples is subpar, so LOOCV accuracy estimates from mouse data may not transfer
- The within-sample normalization step assumes gene-pair differences depend solely on circadian time, which could bias fold-wise predictions in strongly perturbed systems
- Choosing the top ten diurnal genes does not guarantee the best performance, though it is a safe and reasonable choice across datasets

## Evidence

- [results] In leave-one-out cross-validation within the disturbed systems, tauFisher produced high accuracy (feeding ZT5-ZT9: 0.875; feeding ZT0-ZT4: 1) and low RMSE (feeding ZT5-ZT9: 2.236; feeding ZT0-ZT4: 1.061): [evidence span withheld — anchor status: not_found]
- [discussion] calculating and scaling the difference in expression for each pair of predictor genes: "calculating and scaling the difference in expression for each pair of predictor genes"
- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [discussion] It then selects the top ten statistically significant genes with a 24-hour period length.: "It then selects the top ten statistically significant genes with a 24-hour period length."
- [readme] train_tauFisher requires R >=3.5.0 and has been tested on R 4.0.5: [evidence span withheld — anchor status: external_doc]
