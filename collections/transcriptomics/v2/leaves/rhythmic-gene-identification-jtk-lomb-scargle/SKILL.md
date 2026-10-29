---
name: rhythmic-gene-identification-jtk-lomb-scargle
description: Use when a bulk RNAseq/microarray time series with known circadian timepoints
  needs candidate rhythmic genes before training a circadian-time predictor such as
  tauFisher.
when_to_use_negative:
- Input is single-cell-level data without pseudobulk aggregation — high scRNAseq dropout
  prevents reliable rhythm detection and tauFisher cannot predict at single-cell level
- The time series lacks known circadian timestamps and fewer than ~2 full cycles,
  so neither JTK_Cycle nor Lomb-Scargle can resolve a 24-hour period
- Time points include circadian-disturbed conditions (e.g., the 38-hour continuous
  wakefulness samples in GSE56931) that must first be filtered out
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: tauFisher
  role: R package providing run_metacycle and find_periodic_genes to detect 24-hour
    rhythmic genes and pass them as periodic_genes to train_tauFisher
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Runs JTK_Cycle and Lomb-Scargle (method=c('JTK','LS')) to compute per-gene
    period and p-value statistics for the training time series
- name: fda
  role: 'Downstream of this step: Fourier-basis (numbasis=5) functional data smoothing
    of the rhythmic predictor genes during tauFisher training'
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:05384a9c36943a02f708527c2645b2eaac6e1cb3234859c765a85c70f7232db4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - JTK_Cycle / Lomb-Scargle
  - MetaCycle
  - dplyr
  - fda
  - nnet
  - stats
  - stringr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# Rhythmic gene identification with JTK_CYCLE and Lomb-Scargle

## Summary

Identifies genes with a ~24-hour circadian period from bulk transcriptomic time series using MetaCycle's JTK_Cycle and Lomb-Scargle methods, then retains rhythmic genes that overlap the prediction data as tauFisher predictor features.

## When to use

Use when a bulk RNAseq/microarray time series with known circadian timepoints needs candidate rhythmic genes before training a circadian-time predictor such as tauFisher.

## When NOT to use

- Input is single-cell-level data without pseudobulk aggregation — high scRNAseq dropout prevents reliable rhythm detection and tauFisher cannot predict at single-cell level
- The time series lacks known circadian timestamps and fewer than ~2 full cycles, so neither JTK_Cycle nor Lomb-Scargle can resolve a 24-hour period
- Time points include circadian-disturbed conditions (e.g., the 38-hour continuous wakefulness samples in GSE56931) that must first be filtered out

## Inputs

- Normalized bulk transcriptomic time-series matrix (microarray or RNAseq) with circadian timepoints in column names
- Vector of (possibly replicate-offset) circadian timepoints aligned to matrix columns
- Optional test-data gene list for intersecting rhythmic genes with features available at prediction time

## Outputs

- MetaCycle output file (e.g., GSE157077_full_adj.txt) with per-gene JTK and Lomb-Scargle period and p-value statistics
- List of rhythmic genes per method (e.g., genes$JTK) suitable as periodic_genes input to train_tauFisher

## How to apply

After preprocessing the training matrix (uppercase/averaged duplicate gene IDs, timepoints parsed from column names, replicates offset by 24 h so columns are ordered in time), call run_metacycle with method=c('JTK','LS') so both JTK_Cycle and Lomb-Scargle test each gene for a 24-hour period. Then call find_periodic_genes on the MetaCycle output to extract per-method rhythmic gene sets; in the GSE38622 skin-microarray benchmark a JTK p-value < 0.05 cutoff was used, while the GSE157077 SCN benchmark used thres=1 with no p-value cutoff. Retain only rhythmic genes also present in the test data, since genes absent at prediction time (e.g., lost to single-cell dropout) cannot contribute. The resulting gene list feeds train_tauFisher, which selects predictor genes (e.g., 18 in the skin cross-platform test) from this rhythmic set.

## Related tools

- **tauFisher** (R package providing run_metacycle and find_periodic_genes to detect 24-hour rhythmic genes and pass them as periodic_genes to train_tauFisher) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Runs JTK_Cycle and Lomb-Scargle (method=c('JTK','LS')) to compute per-gene period and p-value statistics for the training time series)
- **fda** (Downstream of this step: Fourier-basis (numbasis=5) functional data smoothing of the rhythmic predictor genes during tauFisher training)

## Examples

```
genes = find_periodic_genes(input_file=meta2d_file, method='JTK'); trained <- train_tauFisher(periodic_genes=genes$JTK, nrep=3, numbasis=5)
```

## Evaluation signals

- MetaCycle output contains per-gene JTK_pvalue (and LS) statistics; genes with JTK_pvalue < 0.05 are called significantly rhythmic
- find_periodic_genes returns non-empty gene lists per method that overlap genes present in the test data
- Rhythmic gene set, when passed to train_tauFisher, yields a small predictor gene set (e.g., 18 genes on the skin cross-platform benchmark)
- On cross-platform skin benchmarks the downstream predictor reaches ~0.75 accuracy and RMSE ~4.7 (within 2 hours of truth), confirming the rhythmic features are informative

## Limitations

- JTK and Lomb-Scargle require replicate-ordered time series; replicate columns must be offset by 24 h before calling run_metacycle
- Threshold choice varies by dataset (JTK p<0.05 vs thres=1 with no cutoff); overly strict cutoffs may exclude useful predictor genes
- Genes rhythmic in training but absent in test data (e.g., A630005I04Rik and Ivl lost to dropout in GSE223109 dermis) cannot be used at prediction time

## Evidence

- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [methods] run_metacycle call with JTK and LS methods: "run_metacycle(df=bulk_adj, timepoints=time_adj, out_file="../inst/extdata/GSE157077_full_adj.txt""
- [discussion] tauFisher identifies rhythmic genes: "identifying a subset of diurnal genes with a period length of 24 hours"
- [readme] tauFisher installation instructions: [evidence span withheld — anchor status: external_doc]
