---
name: single-cell-pseudobulk-count-aggregation
description: Use when you have raw scRNAseq count matrices with circadian (or other
  periodic) time labels and need bulk-like profiles so a predictor trained on bulk
  RNAseq or microarray data can score the samples.
when_to_use_negative:
- 'You need circadian time predictions for individual cells: tauFisher cannot overcome
  the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time
  at a single-cell level'
- The single-cell data are not raw counts (e.g. already SCTransform-normalized/integrated
  matrices), since the aggregation step requires summing raw counts per gene across
  all cells
- Your test samples are diseased or circadian-disturbed and only a healthy-donor training
  time series exists; tauFisher trained on healthy data cannot provide diseased-sample
  timescales
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: 'Downstream consumer of the pseudobulk matrix: train_tauFisher (nrep=3, numbasis=5)
    trains on bulk data and test_tauFisher predicts circadian time of the aggregated
    samples; results scored with calc_error'
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: run_metacycle with method=c('JTK','LS') on the bulk training matrix to identify
    rhythmic genes that become the tauFisher predictors
  repo: https://github.com/micnngo/tauFisher
- name: CellRanger
  role: Upstream processing of raw scRNAseq sequencing output with the MM10 reference
    to produce the count matrices that are aggregated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:a8e1c9c07d7f3612dc20dd2f2e832bff8e19f54019c5f7a07112daa2880ff78d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - CellRanger
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-pseudobulk-count-aggregation

## Summary

Aggregates raw single-cell RNAseq counts by summing per gene across all cells at each circadian time point to build pseudobulk samples, enabling bulk-trained circadian time predictors like tauFisher to be applied to single-cell data despite scRNAseq dropout.

## When to use

Use when you have raw scRNAseq count matrices with circadian (or other periodic) time labels and need bulk-like profiles so a predictor trained on bulk RNAseq or microarray data can score the samples.

## When NOT to use

- You need circadian time predictions for individual cells: tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single-cell level
- The single-cell data are not raw counts (e.g. already SCTransform-normalized/integrated matrices), since the aggregation step requires summing raw counts per gene across all cells
- Your test samples are diseased or circadian-disturbed and only a healthy-donor training time series exists; tauFisher trained on healthy data cannot provide diseased-sample timescales

## Inputs

- Raw scRNAseq count matrix (gene x cell) with per-cell circadian time labels, e.g. GSE117295 mouse SCN or GSE223109 mouse dermis
- Grouping variable: circadian time point per cell (control/time-labeled samples only)
- Optionally, the bulk/microarray training expression matrix whose gene IDs the pseudobulk matrix must match (e.g. GSE157077, GSE38622)

## Outputs

- Pseudobulk count/expression matrix (gene x circadian time point), one aggregate sample per time (e.g. 12 SCN or 18 dermis pseudobulk samples)
- Predicted circadian times, true times, accuracy (fraction within 2 h), and RMSE, e.g. written to tauFisher_pseudobulk_predictions.csv

## How to apply

Load the raw single-cell count matrix (e.g. GSE117295 SCN or GSE223109 dermis, processed via CellRanger 3.1.0 with the MM10 reference) and, with no preprocessing, sum raw counts per gene across all cells collected at each labeled circadian time to form one pseudobulk sample per time point (12 SCN control samples, 18 dermis samples in the paper). Keep gene identifiers consistent with the training matrix; expect some predictor genes to be lost to single-cell dropout (e.g. only 16 trained predictor genes survived for the dermis benchmark, with A630005I04Rik and Ivl dropping out). Feed the resulting pseudobulk matrix into the downstream prediction pipeline (find_periodic_genes-filtered tauFisher predictors, train_tauFisher with nrep=3, numbasis=5, test_tauFisher) and evaluate with calc_error using a within-2-hours accuracy criterion plus RMSE.

## Related tools

- **tauFisher** (Downstream consumer of the pseudobulk matrix: train_tauFisher (nrep=3, numbasis=5) trains on bulk data and test_tauFisher predicts circadian time of the aggregated samples; results scored with calc_error) — https://github.com/micnngo/tauFisher
- **MetaCycle** (run_metacycle with method=c('JTK','LS') on the bulk training matrix to identify rhythmic genes that become the tauFisher predictors) — https://github.com/micnngo/tauFisher
- **CellRanger** (Upstream processing of raw scRNAseq sequencing output with the MM10 reference to produce the count matrices that are aggregated)

## Examples

```
pb <- sapply(split(colnames(sce), sce$circadian_time), function(cl) rowSums(counts(sce)[, cl])); result <- test_tauFisher(chosen_genes=trained_JTK$Genes, df=pb)
```

## Evaluation signals

- SCN benchmark reproduces accuracy 0.833 (10/12 predictions within 2 h of labeled time) and RMSE 1.936 on twelve control GSE117295 pseudobulk samples
- Dermis benchmark reproduces accuracy 0.778 (14/18 within 2 h) and RMSE 2.198 on 18 GSE223109 pseudobulk samples with only 16 of the trained predictor genes surviving dropout
- Pseudobulk matrix has exactly one aggregate column per circadian time point (e.g. 12 for SCN), built by summation of raw counts with no preprocessing applied before aggregation
- Predictions, truth, accuracy, and RMSE for both benchmarks are present in tauFisher_pseudobulk_predictions.csv

## Limitations

- tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single-cell level — aggregation to pseudobulk is required
- Dropout can remove trained predictor genes entirely (A630005I04Rik and Ivl dropped out in the dermis benchmark, leaving 16 of the original features), degrading accuracy (0.778) relative to SCN (0.833)

## Evidence

- [methods] Pseudobulk aggregation of scRNAseq counts: [evidence span withheld — anchor status: not_found]
- [discussion] Dropout of predictor genes in single-cell data: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [methods] Upstream scRNAseq processing: "We used CellRanger version 3.1.0 with MM10 reference to process the raw sequencing output."
- [abstract] Cross-platform validation result on pseudobulk samples: [evidence span withheld — anchor status: not_found]
- [readme] tauFisher package installation and environment: [evidence span withheld — anchor status: external_doc]
