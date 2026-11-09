---
name: cross-platform-circadian-prediction-evaluation
description: Use when a bulk transcriptomic time series with labeled circadian times
  (microarray or RNAseq) must predict circadian time of samples from a different assay
  platform, e.g. RNAseq or scRNAseq pseudobulk, and accuracy/RMSE must be reported.
when_to_use_negative:
- 'You need circadian time at the single-cell level: tauFisher cannot overcome high
  scRNAseq dropout and predicts only on bulk or pseudobulk samples'
- 'The test samples are circadian-disturbed or diseased: tauFisher trained on healthy
  time series cannot project onto a diseased timescale without a diseased training
  series'
- The predictor genes are absent from the test platform/assay beyond dropout tolerance
  (only 16 of 18 genes survived in the dermis benchmark; extensive loss would invalidate
  the model)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: 'Core pipeline: train_tauFisher learns predictor genes, PCA and multinomial
    logistic regression from the training platform; test_tauFisher predicts circadian
    time on the other platform; calc_error scores within-2-hour accuracy and RMSE'
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: run_metacycle with methods JTK and LS detects rhythmic genes in the training
    series; find_periodic_genes selects JTK-significant genes as prediction candidates
- name: fda
  role: Fourier-basis functional data analysis (numbasis=5) smooths discrete training
    time points into continuous expression functions
- name: nnet
  role: Fits the multinomial logistic regression on the first two principal components
    of scaled pairwise gene-expression differences
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:80d51cf3742eab9b6514ac6477e3a5cf063980e2fa91e89426411295aae11d3e
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-platform-circadian-prediction-evaluation

## Summary

Benchmark that trains tauFisher on a bulk transcriptomic circadian time series from one assay platform (e.g. GSE38622 mouse skin microarray) and evaluates whether it predicts circadian time of samples measured on another platform (RNAseq or scRNAseq pseudobulk), scored by within-2-hour accuracy and RMSE.

## When to use

Use when a bulk transcriptomic time series with labeled circadian times (microarray or RNAseq) must predict circadian time of samples from a different assay platform, e.g. RNAseq or scRNAseq pseudobulk, and accuracy/RMSE must be reported.

## When NOT to use

- You need circadian time at the single-cell level: tauFisher cannot overcome high scRNAseq dropout and predicts only on bulk or pseudobulk samples
- The test samples are circadian-disturbed or diseased: tauFisher trained on healthy time series cannot project onto a diseased timescale without a diseased training series
- The predictor genes are absent from the test platform/assay beyond dropout tolerance (only 16 of 18 genes survived in the dermis benchmark; extensive loss would invalidate the model)

## Inputs

- Normalized bulk microarray circadian time series with labeled time points and replicates (e.g. GSE38622 mouse skin)
- Bulk RNAseq count matrix with labeled circadian times (e.g. GSE157077 SCN) or as cross-platform test set (e.g. GSE83855 ad libitum controls)
- scRNAseq count matrices from which pseudobulk samples are formed by summing raw counts per gene across all cells at each circadian time (e.g. GSE117295 SCN, GSE223109 dermis)

## Outputs

- predictions.csv with true circadian time, predicted circadian time, accuracy (within 2 hours) and RMSE
- Trained tauFisher model with 18 predictor genes (e.g. 16 surviving single-cell dropout in the dermis benchmark)
- tauFisher_pseudobulk_predictions.csv table of predicted times, truth, accuracy and RMSE for both benchmarks

## How to apply

Preprocess the training matrix (keep complete cases, upper-case and average non-unique gene IDs, parse circadian times from column names, offset each replicate by 24 h), then run run_metacycle with both JTK_Cycle and Lomb-Scargle and call find_periodic_genes to keep genes rhythmic at JTK p-value < 0.05 that are also present in the test data. Train with train_tauFisher (nrep=3, numbasis=5): log2(x+1) transform, scale pairwise gene-expression differences between predictor genes, smooth each gene with Fourier-basis functional data analysis (K=5), run PCA, and fit a multinomial logistic regression on the first two principal components (18 predictor genes for the GSE38622 skin benchmark). Apply test_tauFisher to the held-out platform samples (e.g. 8 GSE83855 ad libitum RNAseq controls or 18 GSE223109 dermis pseudobulk samples), subsetting to the trained predictor genes and projecting onto the training PCA embedding. Score with calc_error, counting a prediction correct when it is within 2 hours of the true label; the reference benchmarks yield accuracy 0.75 / RMSE 4.704 (skin bulk-to-bulk cross-platform), accuracy 0.833 / RMSE 1.936 (SCN bulk-to-pseudobulk), and accuracy 0.778 / RMSE 2.198 (dermis bulk-to-pseudobulk).

## Related tools

- **tauFisher** (Core pipeline: train_tauFisher learns predictor genes, PCA and multinomial logistic regression from the training platform; test_tauFisher predicts circadian time on the other platform; calc_error scores within-2-hour accuracy and RMSE) — github.com/micnngo/tauFisher
- **MetaCycle** (run_metacycle with methods JTK and LS detects rhythmic genes in the training series; find_periodic_genes selects JTK-significant genes as prediction candidates)
- **fda** (Fourier-basis functional data analysis (numbasis=5) smooths discrete training time points into continuous expression functions)
- **nnet** (Fits the multinomial logistic regression on the first two principal components of scaled pairwise gene-expression differences)

## Examples

```
run_metacycle(df=bulk_adj, timepoints=time_adj, out_file="meta2d_full.txt"); genes <- find_periodic_genes(input_file="meta2d_full.txt"); trained <- train_tauFisher(periodic_genes=genes$JTK, nrep=3, numbasis=5); res <- test_tauFisher(chosen_genes=trained$Genes)
```

## Evaluation signals

- On the GSE38622 (microarray) → GSE83855 (RNAseq) benchmark, accuracy is 0.75 and RMSE 4.704: 6 of 8 ad libitum test predictions within 2 hours of truth
- SCN benchmark (GSE157077 bulk → 12 GSE117295 pseudobulk) reproduces accuracy 0.833 and RMSE 1.936; dermis benchmark (GSE38622 → 18 GSE223109 pseudobulk) reproduces accuracy 0.778
- predictions.csv / tauFisher_pseudobulk_predictions.csv contain per-sample true time, predicted time, and the aggregate accuracy and RMSE columns
- Exactly 18 predictor genes from training (16 retained after dermis pseudobulk dropout), selected as rhythmic genes with JTK_pvalue < 0.05 present in the test data
- One-sample test predictions match TimeSignatR-style accuracy and RMSE even though TimeSignatR requires two test samples

## Limitations

- Performance on human blood samples is subpar despite strong results on mouse benchmark datasets
- No confidence scores or uncertainty quantification are output; imputation of predictor gene expression is unimplemented future work
- Within-sample normalization assumes gene-pair expression differences depend solely on circadian time
- Alternative ordinal regression models were not robust and failed on the full set of time points, so multinomial regression is required

## Evidence

- [abstract] tauFisher is robust and generalizable across assay platforms, experimental setups, and tissue types: "Given its robustness and generalizability across assay platforms, experimental setups, and tissue types"
- [discussion] JTK rhythmicity cutoff for predictor selection: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [discussion] Within-sample normalization by pairwise differences: "calculating and scaling the difference in expression for each pair of predictor genes"
- [discussion] PCA plus multinomial regression prediction step: "linearly transforming the scaled differences using principal component analysis and fitting a multinomial logistic regression on the first two principal components"
- [discussion] Comparison to TimeSignatR with one test sample: "tauFisher using one test sample achieved similar accuracy and RMSE for most of the datasets when compared to TimeSignatR which required two test samples"
- [readme] tauFisher R package dependencies: [evidence span withheld — anchor status: not_found]
