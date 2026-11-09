---
name: circadian-time-series-preprocessing
description: Use when a raw bulk transcriptomic time-series matrix with time-encoded
  column names must be converted into tauFisher training/testing inputs — e.
when_to_use_negative:
- You only have unsorted single timepoint samples with no rhythmic-gene training series
  — tauFisher training requires a time series with known circadian times; diseased-sample
  projection additionally needs a time series from diseased individuals
- You need single-cell-level circadian time prediction — scRNAseq dropout prevents
  tauFisher from predicting at single-cell resolution (use pseudobulk)
- Your data are human blood samples — tauFisher's performance there is reported as
  subpar
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: tauFisher
  role: R package providing run_metacycle, find_periodic_genes, get_FDAcurves, create_DiffMatrix,
    scale_DiffMatrix, train_tauFisher, and test_tauFisher for preprocessing and modeling
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Runs JTK_Cycle and Lomb-Scargle rhythmicity detection (meta2d) to identify
    24-hour periodic genes
- name: fda
  role: Functional data analysis smoothing of gene time-expression curves with a Fourier
    basis (numbasis=5)
- name: stats
  role: PCA via stats::prcomp on the scaled pairwise-difference features
- name: nnet
  role: Multinomial logistic regression (nnet::multinom) on the first two principal
    components
- name: dplyr
  role: Data manipulation during preprocessing of expression matrices
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:ac3cdc99171c3c04f755941cc16f0ab5d84246fcde65d3d571480c955847545a
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
  - tauFisher
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# circadian-time-series-preprocessing

## Summary

Normalize and harmonize transcriptomic circadian time series (e.g., GEO microarray/RNAseq matrices like GSE38622 and GSE83855) into the tauFisher-ready form: uppercase/average duplicate gene IDs, order and wrap columns by circadian time, log2(x+1) transform, smooth with functional data analysis (Fourier basis, numbasis=5), and compute scale-normalized pairwise gene-expression differences that enable platform-independent, within-sample circadian time prediction.

## When to use

Use when a raw bulk transcriptomic time-series matrix with time-encoded column names must be converted into tauFisher training/testing inputs — e.g., GEO expression matrices (GSE38622, GSE83855, GSE157077) needing gene-ID harmonization, timepoint reordering, log2 transformation, and pairwise-difference normalization.

## When NOT to use

- You only have unsorted single timepoint samples with no rhythmic-gene training series — tauFisher training requires a time series with known circadian times; diseased-sample projection additionally needs a time series from diseased individuals
- You need single-cell-level circadian time prediction — scRNAseq dropout prevents tauFisher from predicting at single-cell resolution (use pseudobulk)
- Your data are human blood samples — tauFisher's performance there is reported as subpar

## Inputs

- Bulk transcriptomic time-series expression matrix (microarray or RNAseq, e.g., GSE38622 microarray, GSE83855 RNAseq, GSE157077) with circadian times encoded in column names
- Known circadian time labels / replicate indices per sample
- Optional list of rhythmic genes from run_metacycle output (meta2d file) and core clock genes

## Outputs

- Time-ordered, duplicate-averaged, log2(x+1)-transformed expression matrix restricted to rhythmic predictor genes (~18 genes)
- FDA-smoothed gene time courses (Fourier basis, numbasis=5)
- Scaled pairwise gene-expression difference matrix (scaled to [0,1]) ready for PCA and multinomial regression
- Serialized trained model artifact (predictor genes, PCA embedding, fitted model) plus predictor gene list

## How to apply

First preprocess the expression matrix: keep complete cases, upper-case and average non-unique gene IDs, parse circadian times and replicate indices from column names, and offset each replicate by 24 h so all columns are ordered in time (wrapping times to 24 h). Identify rhythmic predictor genes by running run_metacycle (MetaCycle meta2d with JTK_Cycle and Lomb-Scargle) and retaining genes with JTK p-value < 0.05 that are also present in the test data, selecting the top ten statistically significant 24-hour period genes plus core clock genes. Then log2-transform the subset as log2(x+1), smooth each gene's time course with functional data analysis using a Fourier basis with numbasis=5 (get_FDAcurves), compute all pairwise gene differences with create_DiffMatrix, and rescale them to [0,1] with scale_DiffMatrix so the features feed PCA (stats::prcomp, scale=FALSE) and multinomial regression (nnet::multinom on the first two principal components). This within-sample normalization is what makes predictions robust across assay platforms and is the rationale for operating on pairwise differences rather than raw expression values.

## Related tools

- **tauFisher** (R package providing run_metacycle, find_periodic_genes, get_FDAcurves, create_DiffMatrix, scale_DiffMatrix, train_tauFisher, and test_tauFisher for preprocessing and modeling) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Runs JTK_Cycle and Lomb-Scargle rhythmicity detection (meta2d) to identify 24-hour periodic genes)
- **fda** (Functional data analysis smoothing of gene time-expression curves with a Fourier basis (numbasis=5))
- **stats** (PCA via stats::prcomp on the scaled pairwise-difference features)
- **nnet** (Multinomial logistic regression (nnet::multinom) on the first two principal components)
- **dplyr** (Data manipulation during preprocessing of expression matrices)

## Examples

```
genes = find_periodic_genes(input_file=meta2d_file, ...); trained <- train_tauFisher(periodic_genes=genes$JTK, ...)
```

## Evaluation signals

- Preprocessed matrix has upper-cased, averaged gene IDs and columns strictly ordered in adjusted circadian time (replicates offset by 24 h)
- Rhythmic-gene filter retains genes with JTK_pvalue < 0.05 and yields the expected predictor set (~18 genes for GSE38622: top-ten 24-hour genes + 9 core clock genes)
- Scaled difference features fall in [0,1]; PCA is fit on training differences only and test samples are projected onto the training embedding
- Trained RData artifact contains all three components (predictor genes, PCA embedding, fitted model) and predictions.csv contains true time, predicted time, accuracy, and RMSE
- Cross-platform check: GSE38622-trained model achieves accuracy 0.75 and RMSE 4.704 on GSE83855 ad libitum samples with correct predictions defined as within 2 hours of truth

## Limitations

- Within-sample normalization rests on an unverified assumption that gene-pair expression differences depend solely on circadian time
- Choosing the top ten diurnal genes does not guarantee optimal performance, though it is a safe, reasonable choice across datasets
- tauFisher's performance on human blood samples is subpar, and high scRNAseq dropout prevents single-cell-level prediction

## Evidence

- [discussion] Functional data analysis with Fourier basis expansion (K=5) to smooth time-expression curves: "tauFisher uses functional data analysis to represent the discrete time points as continuous functions"
- [discussion] Log2 transformation and pairwise difference scaling: "X′ is then log-transformed element-wise"
- [discussion] Pairwise difference computation is the within-sample normalization core: "calculating and scaling the difference in expression for each pair of predictor genes"
- [discussion] Rhythmicity threshold for predictor selection: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [methods] run_metacycle invocation pattern from the methods: "run_metacycle(df=bulk_adj, 
              timepoints=time_adj, 
              out_file="../inst/extdata/GSE157077_full_adj.txt","
