---
name: sc-rna-seq-circadian-phase-inference
description: Use when scRNA-seq data (droplet-based like 10X Genomics Chromium or
  plate-based) lack sample-time labels and circadian phase per cell must be estimated
  from expression of core clock genes plus de novo-identified cyclers.
when_to_use_negative:
- When sample collection times are known and trusted — the supervised assumption that
  sampling time proxies circadian phase may suffice and unsupervised inference is
  unnecessary
- When the goal is cell-cycle (not circadian) phase ordering, since existing cell-cycle
  methods were designed for that and Tempo targets the ~20-gene circadian clock
- When data are collected as multiple samples over time requiring explicit technical-effect
  modeling, which Tempo does not model
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Bayesian variational inference algorithm performing the circadian phase inference
    and de novo cycler detection
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Python package for handling the single-cell expression matrix input
  repo: https://github.com/bauerbach95/tempo
- name: scanpy>=1.6
  role: Python dependency for single-cell preprocessing in the Tempo environment
  repo: https://github.com/bauerbach95/tempo
- name: pytorch>=1.9.0
  role: Deep-learning backend supporting the variational inference implementation
  repo: https://github.com/bauerbach95/tempo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:b40f679359b4e51ab026b302d4a5125a83caf5fdbf01652b7535c7b030c58de0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch>=1.9.0
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sc-rna-seq-circadian-phase-inference

## Summary

Unsupervised Bayesian variational inference of circadian cell phase from single-cell RNA-seq data using Tempo, which jointly fits cell phases, cycler parameters, and de novo clock-controlled genes while quantifying phase-estimation uncertainty.

## When to use

Use when scRNA-seq data (droplet-based like 10X Genomics Chromium or plate-based) lack sample-time labels and circadian phase per cell must be estimated from expression of core clock genes plus de novo-identified cyclers.

## When NOT to use

- When sample collection times are known and trusted — the supervised assumption that sampling time proxies circadian phase may suffice and unsupervised inference is unnecessary
- When the goal is cell-cycle (not circadian) phase ordering, since existing cell-cycle methods were designed for that and Tempo targets the ~20-gene circadian clock
- When data are collected as multiple samples over time requiring explicit technical-effect modeling, which Tempo does not model

## Inputs

- scRNA-seq expression matrix as an anndata object (droplet- or plate-based)
- User-supplied list of core circadian clock genes
- Prior distributions for model parameters

## Outputs

- Per-cell circadian phase point estimates with uncertainty quantification
- Optimized approximate joint posterior q(theta, beta)
- Set of de novo-identified cycling genes (CCGs)

## How to apply

Install Tempo via git clone and source install.sh (requires mini-conda, python >= 3.8, pytorch>=1.9.0, anndata, numpy, pandas, scanpy>=1.6), then activate the 'tempo' conda environment. Supply core clock genes and prior distributions along with the expression matrix (anndata object); Tempo minimizes KL divergence between the approximate joint posterior q(theta, beta) and the true posterior to jointly fit cell phases and cycler parameters (Step 1), then detects de novo cyclers and fits non-cycling gene parameters conditional on the phase estimates (Step 2). These steps iterate until the core clock gene Bayesian evidence worsens relative to the prior iteration or falls below a random-baseline evidence, or a maximum iteration count is exceeded; the output is the optimized approximate joint posterior q(theta, beta) with phase-uncertainty quantification. Correct application is judged by a run_test.py run completing under 5 minutes with the 'SUCCESSFULLY FINISHED' message, and by well-calibrated phase estimates versus existing methods.

## Related tools

- **Tempo** (Bayesian variational inference algorithm performing the circadian phase inference and de novo cycler detection) — https://github.com/bauerbach95/tempo
- **anndata** (Python package for handling the single-cell expression matrix input) — https://github.com/bauerbach95/tempo
- **scanpy>=1.6** (Python dependency for single-cell preprocessing in the Tempo environment) — https://github.com/bauerbach95/tempo
- **pytorch>=1.9.0** (Deep-learning backend supporting the variational inference implementation) — https://github.com/bauerbach95/tempo

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- python run_test.py in the activated tempo conda environment completes in under 5 minutes and prints 'SUCCESSFULLY FINISHED'
- Iteration terminates only when core clock gene Bayesian evidence worsens versus the prior iteration or falls below a random-baseline evidence, or maximum iterations are exceeded
- Phase estimates are more accurate than existing unsupervised methods and uncertainties are well calibrated
- Runtime scales acceptably with cell number (Tempo was designed to fix poor scaling of prior methods on droplet-based data)

## Limitations

- Assumes CCG expression means follow 24 h sinusoidal patterns across the circadian cycle, which simulations suggest may misclassify some de novo cyclers; more flexible waveform modeling is needed
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell-state embedding to sinusoidal parameters would be an ideal solution
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, and is not built in
- Requires python >= 3.8 and >=8GB RAM for in-memory operations; tested only on macOS 10.14.5/12.2.1 and CentOS Linux 7

## Evidence

- [methods] Tempo repeats Step 1 ... and Step 2 (de novo cycler detection) until the core clock gene Bayesian evidence worsens or the maximum number of iterations is exceeded: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [readme] Environment and dependency setup: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Validation test expectation: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [abstract] Performance claim supporting correct application: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [discussion] Discussion limitation on sinusoidal assumption: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
