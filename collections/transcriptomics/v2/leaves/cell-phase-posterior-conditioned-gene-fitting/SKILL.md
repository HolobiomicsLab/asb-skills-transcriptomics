---
name: cell-phase-posterior-conditioned-gene-fitting
description: Use when Step 1 phase posteriors exist for cells in a droplet/plate scRNA-seq
  dataset and you need to identify de novo circadian cycling genes among currently
  non-cycling genes before refitting the model.
when_to_use_negative:
- Genes are already designated as cyclers — this skill specifically targets genes
  not currently identified as cyclers for de novo selection
- No Step 1 cell phase posterior distributions are available, since the gene fits
  are conditioned on those phase posteriors
- Flexible non-sinusoidal waveform modeling is required — the article notes the de
  novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Tempo
  role: Bayesian variational inference framework whose Step 2 module fits gene amplitude
    distributions conditioned on Step 1 phase posteriors
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container for the single-cell scRNA-seq expression matrix input
- name: PyTorch
  role: Backend for the variational inference fits of approximate gene parameter distributions
- name: numpy
  role: Numerical computation of MAP amplitude statistics and Pearson residuals
- name: pandas
  role: Writes the retained gene list with filter-statistic columns to cycler_genes.csv
- name: scipy
  role: Statistical computations for the Pearson residual criterion
- name: statsmodels
  role: Statistical modeling support for the gene fitting step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:b99cc8d7dcf80f03923c3827bc04369da556cd434f4ff138444d689a1e29d140
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-phase-posterior-conditioned-gene-fitting

## Summary

Fits approximate sinusoidal amplitude parameter distributions for genes not currently designated cyclers, conditioned on Step 1 cell phase posterior distributions, and selects de novo cycling genes using the MAP amplitude Pearson residual and non-zero-amplitude MAP fraction criteria (Tempo Step 2).

## When to use

Use when Step 1 phase posteriors exist for cells in a droplet/plate scRNA-seq dataset and you need to identify de novo circadian cycling genes among currently non-cycling genes before refitting the model.

## When NOT to use

- Genes are already designated as cyclers — this skill specifically targets genes not currently identified as cyclers for de novo selection
- No Step 1 cell phase posterior distributions are available, since the gene fits are conditioned on those phase posteriors
- Flexible non-sinusoidal waveform modeling is required — the article notes the de novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means

## Inputs

- Single-cell scRNA-seq expression matrix as an anndata object
- Cell phase posterior distributions from Tempo Step 1 (used as point phase estimates with Q_j forced to 1)

## Outputs

- cycler_genes.csv containing a gene identifier column plus per-gene MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns
- Expanded set of current cycling genes including the newly selected de novo cyclers

## How to apply

Load the scRNA-seq expression matrix (anndata object) and the Step 1 cell phase posterior, forcing each cell's phase posterior Q_j to 1 so the gene fits are conditioned on point phase estimates. For every gene currently labeled non-cycling, fit the approximate sinusoidal amplitude parameter distribution implied by the observed expression pattern across the ordered phase posterior. For each gene compute two statistics: the MAP amplitude Pearson residual (deviation of MAP amplitude from its expectation given MAP mesor, as plotted in Fig. 2c) and the fraction of posterior samples with non-zero amplitude. Apply the cycler selection thresholds from Fig. 2c / Supplementary Methods 8 to classify genes as de novo cyclers, then add them to the set of current cycling genes and write the retained gene list with both statistic columns to cycler_genes.csv.

## Related tools

- **Tempo** (Bayesian variational inference framework whose Step 2 module fits gene amplitude distributions conditioned on Step 1 phase posteriors) — https://github.com/bauerbach95/tempo
- **anndata** (Container for the single-cell scRNA-seq expression matrix input)
- **PyTorch** (Backend for the variational inference fits of approximate gene parameter distributions)
- **numpy** (Numerical computation of MAP amplitude statistics and Pearson residuals)
- **pandas** (Writes the retained gene list with filter-statistic columns to cycler_genes.csv)
- **scipy** (Statistical computations for the Pearson residual criterion)
- **statsmodels** (Statistical modeling support for the gene fitting step)

## Examples

```
conda activate tempo && python step2_fit_noncycler_genes.py --anndata expression.h5ad --phase_posterior step1_posteriors.csv --out cycler_genes.csv
```

## Evaluation signals

- cycler_genes.csv exists and contains a gene identifier column plus the two filter-statistic columns (MAP amplitude Pearson residual, non-zero-amplitude MAP fraction)
- Every retained gene satisfies both selection thresholds from Fig. 2c / Supplementary Methods 8
- All non-zero-amplitude MAP fraction values lie between 0 and 1
- Gene fits are conditioned on point phase estimates, verifiable by checking that each cell's phase posterior Q_j was forced to 1 before fitting
- Genes selected as de novo cyclers are those best described by phase variation among the previously non-cycling gene set

## Limitations

- Simulated analyses suggest the de novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means; more flexible waveform modeling is needed
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell state embedding to sinusoidal parameters would be an ideal solution
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, which this conditioning step does not address

## Evidence

- [supplementary] Step 2 conditions on step 1 cell phase posteriors: [evidence span withheld — anchor status: not_found]
- [supplementary] Fig 2c filter criteria: [evidence span withheld — anchor status: not_found]
- [supplementary] Point phase conditioning: [evidence span withheld — anchor status: not_found]
- [readme] README dependencies: "Tempo requires python >= 3.8 and depends on the following python packages: - anndata - numpy - pandas - scanpy>=1.6"
- [discussion] Sinusoidal assumption limitation: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
