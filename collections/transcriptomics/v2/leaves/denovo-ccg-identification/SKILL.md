---
name: denovo-ccg-identification
description: Use when after Tempo Step 1 has produced cell phase posterior distributions
  for a scRNA-seq dataset, you must expand the circadian cycler set by identifying
  which previously non-cycling genes are described by phase variation.
when_to_use_negative:
- When the goal is to refine phases rather than discover de novo cyclers — this Step
  2 module only assesses genes not currently identified as cyclers; already-designated
  cycling genes are not refiltered here.
- When Step 1 cell phase posterior distributions are unavailable or unreliable (e.g.,
  poor phase estimates on droplet-based scRNA-seq), since the gene fit is conditioned
  on the Step 1 phase estimates.
- When a CCG is expected to have strongly non-sinusoidal expression, since the article
  notes the de novo cycler identification may be limited by assuming 24 h sinusoidal
  CCG expression means.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: core algorithm whose Step 2 module performs the de novo cycler fit and MAP-based
    gene selection
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: holds the single-cell scRNA-seq expression matrix input to the gene fit
- name: numpy
  role: numerical computation of the MAP amplitude Pearson residual and amplitude
    posterior statistics
- name: pandas
  role: writes the retained gene list with filter statistics to cycler_genes.csv
- name: PyTorch
  role: variational fitting of the approximate sinusoidal amplitude parameter distributions
    per gene
- name: scipy
  role: dependency of the Tempo environment used in the gene-level statistical computations
- name: statsmodels
  role: dependency of the Tempo environment used in the gene-level statistical computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:453036da3a8a39e8b8cfffce544eae31058f92c61c720661af60529269cc2f44
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# denovo-ccg-identification

## Summary

Tempo's Step 2 module that fits approximate sinusoidal amplitude parameter distributions to genes not currently designated as cyclers, conditioned on the Step 1 cell phase posterior distributions, and selects de novo clock-controlled genes (CCGs) using the MAP amplitude Pearson residual and the non-zero-amplitude MAP fraction (Fig. 2c).

## When to use

Use when, after Tempo Step 1 has produced cell phase posterior distributions for a scRNA-seq dataset, you must expand the circadian cycler set by identifying which previously non-cycling genes are best described by phase variation.

## When NOT to use

- When the goal is to refine phases rather than discover de novo cyclers — this Step 2 module only assesses genes not currently identified as cyclers; already-designated cycling genes are not refiltered here.
- When Step 1 cell phase posterior distributions are unavailable or unreliable (e.g., poor phase estimates on droplet-based scRNA-seq), since the gene fit is conditioned on the Step 1 phase estimates.
- When a CCG is expected to have strongly non-sinusoidal expression, since the article notes the de novo cycler identification may be limited by assuming 24 h sinusoidal CCG expression means.

## Inputs

- single-cell scRNA-seq expression matrix as an anndata object
- cell phase posterior distributions from Tempo Step 1 (collapsed to point phase estimates, Q_j forced to 1)
- list of genes currently designated as non-cycling
- cycler selection thresholds from Fig. 2c / Supplementary Methods 8

## Outputs

- cycler_genes.csv containing a gene identifier column plus per-gene MAP amplitude Pearson residual and non-zero-amplitude MAP fraction columns
- updated set of cycling genes (current cyclers plus newly selected de novo cyclers)

## How to apply

Load the scRNA-seq expression matrix (anndata object) together with the Step 1 cell phase posterior, forcing each cell's phase posterior Q_j to 1 so the gene fit is conditioned on the point phase estimates. For every gene currently labeled non-cycling, fit the approximate sinusoidal amplitude parameter distribution implied by the observed expression pattern across the ordered phase posterior (the variational fit uses PyTorch; numpy/pandas/scipy/statsmodels support the computations). For each gene, compute two filter statistics: the MAP amplitude Pearson residual (deviation of the MAP amplitude from its expectation given the MAP mesor) and the MAP fraction of samples with non-zero amplitude. Apply the cycler selection thresholds of Fig. 2c / Supplementary Methods 8 to these two statistics to classify genes as de novo cyclers, then add them to the current cycling-gene set. Write the retained gene list with per-gene Pearson residual and non-zero-amplitude MAP fraction columns to cycler_genes.csv.

## Related tools

- **Tempo** (core algorithm whose Step 2 module performs the de novo cycler fit and MAP-based gene selection) — https://github.com/bauerbach95/tempo
- **anndata** (holds the single-cell scRNA-seq expression matrix input to the gene fit)
- **numpy** (numerical computation of the MAP amplitude Pearson residual and amplitude posterior statistics)
- **pandas** (writes the retained gene list with filter statistics to cycler_genes.csv)
- **PyTorch** (variational fitting of the approximate sinusoidal amplitude parameter distributions per gene)
- **scipy** (dependency of the Tempo environment used in the gene-level statistical computations)
- **statsmodels** (dependency of the Tempo environment used in the gene-level statistical computations)

## Evaluation signals

- cycler_genes.csv exists and contains a gene identifier column plus the two filter-statistic columns (MAP amplitude Pearson residual, non-zero-amplitude MAP fraction)
- every retained gene satisfies both selection thresholds from Fig. 2c / Supplementary Methods 8
- all non-zero-amplitude MAP fraction values lie between 0 and 1
- genes already designated as cyclers in Step 1 are not reclassified by this Step 2 filter — only non-cycling genes are added
- Tempo environment validated via run_test.py completing with 'SUCCESSFULLY FINISHED'

## Limitations

- The de novo cycler identification may stem from assuming 24 h sinusoidal CCG expression means; simulated analyses suggest more flexible waveform modeling is needed.
- Subpopulation-specific gene amplitude variation is not explicitly modeled; the authors suggest a function mapping a cell-state embedding to sinusoidal parameters would be an ideal solution.
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, which this filter does not address.

## Evidence

- [intro] An optimal approach for estimating circadian phase in scRNA-seq should thus identify CCGs de novo: [evidence span withheld — anchor status: not_found]
- [discussion] this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [intro] Many open questions in chronobiology require single-cell resolution, such as the identification of cell-type-specific CCGs and the role of circadian phase in gating cell fate decisions.: "Many open questions in chronobiology require single-cell resolution, such as the identification of cell-type-specific CCGs and the role of circadian phase in gating cell fate decisions."
- [readme] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages:"
- [discussion] a more ideal solution may be to use a function to map continuous measures of cell state (i.e., a low-dimensional embedding) to the sinusoidal parameters.: "a more ideal solution may be to use a function to map continuous measures of cell state (i.e., a low-dimensional embedding) to the sinusoidal parameters."
