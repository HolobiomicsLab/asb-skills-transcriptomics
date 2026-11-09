---
name: scrnaseq-expression-matrix-processing
description: Use when a single-cell scRNA-seq expression matrix (anndata object) and
  Step 1 cell phase posteriors are available and genes not yet labeled cyclers must
  be screened for de novo circadian rhythmicity before being added to the current
  cycler set.
when_to_use_negative:
- Input is the Step 1 phase inference task itself — this skill assumes Step 1 phase
  posteriors already exist and only operates on genes not yet labeled cyclers
- Genes already designated as cyclers are the target — the filter is defined for currently
  non-cycling genes only
- The expression means of the genes of interest are known to follow strongly non-sinusoidal
  24 h waveforms, which Tempo's sinusoidal CCG model may mischaracterize
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Supplies the Step 2 de novo cycler fitting and MAP-based filtering procedure
    applied to the expression matrix
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container format for the loaded single-cell expression matrix and phase posterior
- name: numpy
  role: Numerical computation of amplitude posterior statistics and Pearson residuals
- name: pandas
  role: Writes cycler_genes.csv with gene identifiers and the two filter-statistic
    columns
- name: scipy
  role: Distribution fitting / statistical computations for the sinusoidal amplitude
    parameter posterior
- name: statsmodels
  role: Regression statistics used for the Pearson residual computation
- name: PyTorch
  role: Backend for Tempo's Bayesian variational inference gene parameter fits (pytorch>=1.9.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:178685085f73ef3f9b4c24cabfaff05e7bf783f80f00887c330397fc11742803
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-expression-matrix-processing

## Summary

Screens genes not currently designated as circadian cyclers using the Tempo Step 2 de novo cycler filter, which fits gene-level sinusoidal amplitude posterior distributions conditioned on Step 1 cell phase posteriors and selects cyclers by the MAP amplitude Pearson residual and non-zero-amplitude MAP fraction statistics (Fig. 2c).

## When to use

Use when a single-cell scRNA-seq expression matrix (anndata object) and Step 1 cell phase posteriors are available and genes not yet labeled cyclers must be screened for de novo circadian rhythmicity before being added to the current cycler set.

## When NOT to use

- Input is the Step 1 phase inference task itself — this skill assumes Step 1 phase posteriors already exist and only operates on genes not yet labeled cyclers
- Genes already designated as cyclers are the target — the filter is defined for currently non-cycling genes only
- The expression means of the genes of interest are known to follow strongly non-sinusoidal 24 h waveforms, which Tempo's sinusoidal CCG model may mischaracterize

## Inputs

- Single-cell scRNA-seq expression matrix as an anndata (.h5ad) object
- Cell phase posterior distributions from Tempo Step 1 (collapsed to point phase estimates by forcing Q_j = 1)
- List of genes currently designated as cyclers (so that only non-cycling genes are screened)
- Cycler selection thresholds from Fig. 2c / Supplementary Methods 8

## Outputs

- cycler_genes.csv containing a gene identifier column plus per-gene MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns
- Expanded set of current cycling genes including newly selected de novo cyclers

## How to apply

Load the scRNA-seq expression matrix (anndata object) together with the cell phase posterior from Step 1, and force each cell's phase posterior Q_j to 1 so the Step 2 gene fit is conditioned on point phase estimates. For every gene currently labeled non-cycling, fit the approximate sinusoidal amplitude parameter distribution implied by the observed expression pattern across the ordered phase posterior, using the Tempo package (installed in the 'tempo' conda environment with Python >= 3.8, anndata, numpy, pandas, scipy, statsmodels, and pytorch>=1.9.0). Per gene, compute the MAP amplitude Pearson residual — the deviation of MAP amplitude from its expectation given the MAP mesor — and the non-zero-amplitude MAP fraction from the fitted amplitude posterior. Apply the cycler selection thresholds shown in Fig. 2c and detailed in Supplementary Methods 8 to these two statistics to classify genes as de novo cyclers. Write the retained gene list with per-gene MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns to cycler_genes.csv; the rationale is that an optimal circadian phase inference approach should identify cell-type-specific clock-controlled genes (CCGs) de novo rather than relying on pre-existing cycler annotations.

## Related tools

- **Tempo** (Supplies the Step 2 de novo cycler fitting and MAP-based filtering procedure applied to the expression matrix) — https://github.com/bauerbach95/tempo
- **anndata** (Container format for the loaded single-cell expression matrix and phase posterior)
- **numpy** (Numerical computation of amplitude posterior statistics and Pearson residuals)
- **pandas** (Writes cycler_genes.csv with gene identifiers and the two filter-statistic columns)
- **scipy** (Distribution fitting / statistical computations for the sinusoidal amplitude parameter posterior)
- **statsmodels** (Regression statistics used for the Pearson residual computation)
- **PyTorch** (Backend for Tempo's Bayesian variational inference gene parameter fits (pytorch>=1.9.0))

## Evaluation signals

- cycler_genes.csv exists and contains a gene identifier column plus the MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns
- Every retained gene satisfies both Fig. 2c / Supplementary Methods 8 cycler selection thresholds
- All non-zero-amplitude MAP fraction values lie between 0 and 1
- No gene already designated a cycler appears among the newly classified de novo cyclers
- The environment passes the repository sanity check: running run_test.py completes with the message 'SUCCESSFULLY FINISHED'

## Limitations

- Tempo's CCG model assumes expression means follow 24 h sinusoidal patterns; simulated analyses suggest this may underlie the de novo cycler issue, and more flexible waveform modeling may be needed
- Subpopulation-specific gene amplitude variation is not explicitly modeled; the authors suggest a function mapping a low-dimensional cell state embedding to sinusoidal parameters as an ideal future solution
- Forcing Q_j to 1 conditions the fit on point phase estimates, discarding the Step 1 phase posterior uncertainty this filter step would otherwise condition on

## Evidence

- [other] Step 2 de novo cycler selection criterion: [evidence span withheld — anchor status: not_found]
- [other] MAP statistics used for classification: [evidence span withheld — anchor status: not_found]
- [readme] Runtime environment and dependencies: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Repository-level validation signal: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [readme] Pointers to input/output specifications for running the filter: "Tutorials on how to run Tempo (and information on inputs / outputs) can be viewed in the tutorial folder of the repository."
