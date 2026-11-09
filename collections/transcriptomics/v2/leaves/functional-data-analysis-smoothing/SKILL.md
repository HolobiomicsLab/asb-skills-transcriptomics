---
name: functional-data-analysis-smoothing
description: Use when training tauFisher on a bulk transcriptomic time course (e.
when_to_use_negative:
- Input is single-cell expression data at the individual-cell level — tauFisher cannot
  overcome the high scRNAseq dropout rate and only works on bulk or pseudobulk profiles
- You need to predict on untrained time scales of diseased samples without a diseased
  time series for training — the smoothed healthy-time reference cannot label them
- The genes of interest have not been assessed for rhythmicity — first run run_metacycle
  and find_periodic_genes (JTK_pvalue < 0.05) rather than smoothing an arbitrary gene
  set
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: tauFisher
  role: Pipeline whose train_tauFisher function performs the FDA smoothing (numbasis=5
    Fourier basis) before scaled pairwise differences, PCA, and multinomial regression
  repo: https://github.com/micnngo/tauFisher
- name: fda
  role: R package supplying the functional data analysis machinery (Fourier basis
    expansions) used by get_FDAcurves to turn discrete time points into continuous
    functions
- name: MetaCycle
  role: Identifies the rhythmic (JTK p-value < 0.05, 24-hour period) genes whose curves
    are subsequently smoothed
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:46100df12bcc58ab93fca9471275fe71fb77ac20384ec05480b8c3cec738d195
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# functional-data-analysis-smoothing

## Summary

Smooths discrete time-point gene-expression curves as continuous functions via functional data analysis with a Fourier basis (numbasis=5), the representation step inside tauFisher's train_tauFisher that precedes pairwise-difference scaling, PCA, and multinomial logistic regression.

## When to use

Use when training tauFisher on a bulk transcriptomic time course (e.g., GSE38622 microarray or GSE83855 RNAseq): discrete circadian time points must be converted to smooth continuous curves (log2(x+1) expression, Fourier basis, numbasis=5) before computing scaled pairwise gene-expression differences.

## When NOT to use

- Input is single-cell expression data at the individual-cell level — tauFisher cannot overcome the high scRNAseq dropout rate and only works on bulk or pseudobulk profiles
- You need to predict on untrained time scales of diseased samples without a diseased time series for training — the smoothed healthy-time reference cannot label them
- The genes of interest have not been assessed for rhythmicity — first run run_metacycle and find_periodic_genes (JTK_pvalue < 0.05) rather than smoothing an arbitrary gene set

## Inputs

- Expression matrix of periodic predictor genes (genes as rows, time-point samples as columns) after log2(x+1) transformation
- Circadian time labels parsed from sample column names (e.g., ZT times, with replicates offset by 24 h so columns are ordered in time)
- Periodic gene list from find_periodic_genes (JTK_pvalue < 0.05, top ten 24-hour period genes)

## Outputs

- Smoothed continuous time-expression curves (get_FDAcurves output, Fourier basis numbasis=5)
- Scaled pairwise gene-expression difference matrix feeding PCA and the nnet::multinom predictor-gene model (e.g., 18 predictor genes in the GSE38622 training)

## How to apply

Inside train_tauFisher, first log2-transform the rhythmic predictor-gene expression matrix (genes selected at JTK p-value < 0.05 by find_periodic_genes after run_metacycle with JTK_Cycle and Lomb-Scargle). Then use functional data analysis (R package fda, via get_FDAcurves) to represent each gene's discrete time points as a continuous function using a Fourier basis with numbasis=5. Compute and scale the pairwise gene-expression differences from the smoothed curves, project them with stats::prcomp PCA, and fit a multinomial logistic regression (nnet::multinom) on the first two principal components. The rationale is that tauFisher predicts circadian time from a single sample by exploiting correlations between rhythmic genes, so the smoothed continuous representation supplies the stable time-resolved shape from which within-sample gene-pair differences are computed.

## Related tools

- **tauFisher** (Pipeline whose train_tauFisher function performs the FDA smoothing (numbasis=5 Fourier basis) before scaled pairwise differences, PCA, and multinomial regression) — https://github.com/micnngo/tauFisher
- **fda** (R package supplying the functional data analysis machinery (Fourier basis expansions) used by get_FDAcurves to turn discrete time points into continuous functions)
- **MetaCycle** (Identifies the rhythmic (JTK p-value < 0.05, 24-hour period) genes whose curves are subsequently smoothed)

## Examples

```
trained_JTK <- train_tauFisher(periodic_genes=genes$JTK, df=bulk_adj)  # internally: log2 transform, get_FDAcurves with Fourier basis numbasis=5
```

## Evaluation signals

- Smoothed curves reproduce when regenerating with get_FDAcurves on the same log2-transformed input using the Fourier basis with numbasis=5
- Downstream pipeline on GSE38622 microarray training selects the expected 18 predictor genes and predicts GSE83855 ad libitum samples with accuracy 0.75 and RMSE 4.704 (within-2-hour criterion via calc_error)
- Leave-one-out cross-validation on GSE83855 disturbed systems reproduces accuracy 0.875 / RMSE 2.236 (ZT5-ZT9) and accuracy 1.0 / RMSE 1.061 (ZT0-ZT4)
- PCA of the scaled pairwise difference matrix is dominated by the first two principal components used for the multinomial logistic regression, confirming the smoothing produced a usable low-dimensional embedding

## Limitations

- The numbasis=5 Fourier basis is the setting validated across the paper's datasets; alternative basis sizes are not benchmarked
- Smoothing operates on bulk/pseudobulk profiles only — high sequencing dropout in scRNAseq prevents single-cell-level application
- Requires R >=3.5.0 (tested on R 4.0.5) with the fda package; tauFisher has been tested only on macOS and Linux

## Evidence

- [discussion] FDA smoothing is an explicit workflow step of tauFisher: "tauFisher uses functional data analysis to represent the discrete time points as continuous functions"
- [discussion] The smoothing feeds pairwise-difference scaling, PCA, and multinomial regression: "linearly transforming the scaled differences using principal component analysis and fitting a multinomial logistic regression on the first two principal components"
- [readme] R runtime environment and package context for the fda-based smoothing: [evidence span withheld — anchor status: external_doc]
- [methods] Smoothing occurs inside the train_tauFisher step of the pipeline: "trained_JTK <- train_tauFisher(periodic_genes=genes$JTK,"
