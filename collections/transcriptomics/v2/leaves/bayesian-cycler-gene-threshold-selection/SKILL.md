---
name: bayesian-cycler-gene-threshold-selection
description: Use when after Tempo Step 1 phase inference, you must expand the core-clock
  cycler set de novo — i.e., classify which currently non-cycling genes from a scRNA-seq
  anndata matrix are described by 24 h phase variation before refitting the model.
when_to_use_negative:
- Genes are already designated cyclers — this module only assesses genes not currently
  identified as cyclers
- No Step 1 phase posterior is available yet — the amplitude fit is conditioned on
  Step 1 cell phase posterior distributions, so phase inference must precede this
  filter
- The goal is flexible non-sinusoidal waveform detection — the article notes the de
  novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: Bayesian variational inference framework whose Step 2 fits approximate gene
    amplitude parameter distributions for non-cycling genes and applies the Fig. 2c
    cycler selection thresholds
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container for the input single-cell expression matrix and Step 1 phase posterior
- name: numpy
  role: Numerical computation of amplitude posterior statistics and Pearson residuals
- name: pandas
  role: Writes the retained gene list with filter-statistic columns to cycler_genes.csv
- name: scipy
  role: Statistical computations supporting the amplitude fits and residual calculations
- name: PyTorch
  role: Backend for Tempo's variational inference fitting of gene parameter distributions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:e1087cdc5fff9566c5a326bf3072a34efc0e998ee4344203f34f0d09e088c0d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - PyTorch
  - Tempo
  - anndata
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bayesian-cycler-gene-threshold-selection

## Summary

Tempo Step 2 module that identifies de novo circadian cycling genes from genes not currently designated as cyclers, by fitting approximate sinusoidal amplitude parameter distributions conditioned on the Step 1 cell phase posterior and filtering genes on two statistics: the MAP amplitude Pearson residual and the non-zero-amplitude MAP fraction (Fig. 2c criterion).

## When to use

Use when, after Tempo Step 1 phase inference, you must expand the core-clock cycler set de novo — i.e., classify which currently non-cycling genes from a scRNA-seq anndata matrix are best described by 24 h phase variation before refitting the model.

## When NOT to use

- Genes are already designated cyclers — this module only assesses genes not currently identified as cyclers
- No Step 1 phase posterior is available yet — the amplitude fit is conditioned on Step 1 cell phase posterior distributions, so phase inference must precede this filter
- The goal is flexible non-sinusoidal waveform detection — the article notes the de novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means

## Inputs

- Single-cell scRNA-seq expression matrix as an anndata object
- Cell phase posterior distributions from Tempo Step 1
- List of genes currently designated as cyclers (to exclude from the de novo search)

## Outputs

- cycler_genes.csv containing a gene identifier column plus per-gene MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns
- Expanded set of cycling genes (existing cyclers plus selected de novo cyclers)

## How to apply

Load the single-cell scRNA-seq expression matrix (anndata object) and the Step 1 cell phase posterior, forcing each cell's phase posterior Q_j to 1 so the amplitude fit is conditioned on point phase estimates. For every gene currently labeled non-cycling, fit the approximate sinusoidal amplitude parameter distribution implied by the observed expression pattern across the ordered phase posterior. Compute, per gene, the MAP amplitude Pearson residual (deviation of MAP amplitude from its expectation given MAP mesor) and the non-zero-amplitude MAP fraction from the fitted amplitude posterior. Apply the cycler selection thresholds of Fig. 2c / Supplementary Methods 8 to these two statistics; genes passing both are added to the current cycling gene set. Write the retained gene list with the two filter-statistic columns to cycler_genes.csv.

## Related tools

- **Tempo** (Bayesian variational inference framework whose Step 2 fits approximate gene amplitude parameter distributions for non-cycling genes and applies the Fig. 2c cycler selection thresholds) — https://github.com/bauerbach95/tempo
- **anndata** (Container for the input single-cell expression matrix and Step 1 phase posterior)
- **numpy** (Numerical computation of amplitude posterior statistics and Pearson residuals)
- **pandas** (Writes the retained gene list with filter-statistic columns to cycler_genes.csv)
- **scipy** (Statistical computations supporting the amplitude fits and residual calculations)
- **PyTorch** (Backend for Tempo's variational inference fitting of gene parameter distributions)

## Examples

```
python run_step2_filter.py --anndata sc_expr.h5ad --phase_posterior step1_posterior.csv --out cycler_genes.csv
```

## Evaluation signals

- cycler_genes.csv exists and contains a gene identifier column plus the two filter-statistic columns (MAP amplitude Pearson residual and non-zero-amplitude MAP fraction)
- Every retained gene satisfies both selection thresholds from Fig. 2c / Supplementary Methods 8
- All non-zero-amplitude MAP fraction values lie between 0 and 1
- The fit reproduces the paper's criterion: selected genes are those 'best described by phase variation' among previously non-cycling genes
- Each selected gene's MAP amplitude deviates from its expectation given the MAP mesor by no more than the Pearson residual threshold plotted in Fig. 2c

## Limitations

- The filter assumes CCG expression means follow 24 h sinusoidal patterns; simulated analyses suggest this assumption may miss genes with more flexible waveforms
- Thresholds are defined in Supplementary Methods 8 / Fig. 2c; exact numeric cutoffs are not stated in the main text, so the criteria must be retrieved from the supplementary material
- Subpopulation-specific gene amplitude variation is not explicitly modeled, which can affect amplitude estimates for heterogeneous cell states
- Conditioning on point phase estimates (Q_j forced to 1) discards Step 1 phase uncertainty, which could misclassify genes for sparse droplet-based data

## Evidence

- [results] Step 2 de novo cycler selection: [evidence span withheld — anchor status: not_found]
- [results] filter statistics: [evidence span withheld — anchor status: not_found]
- [discussion] sinusoidal assumption: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [discussion] amplitude state not modeled: [evidence span withheld — anchor status: not_found]
- [supplementary] validation thresholds: [evidence span withheld — anchor status: not_found]
- [readme] environment: "Tempo requires python >= 3.8 and depends on the following python packages"
