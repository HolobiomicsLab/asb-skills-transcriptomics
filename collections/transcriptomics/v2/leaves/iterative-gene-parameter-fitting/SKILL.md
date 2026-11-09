---
name: iterative-gene-parameter-fitting
description: Use when estimating single-cell circadian phases unsupervised from scRNA-seq
  expression (e.g., anndata matrices) and unknown clock-controlled genes must be identified
  de novo while quantifying phase uncertainty.
when_to_use_negative:
- The analysis goal assumes sample collection timing proxies circadian phase (time-course
  designs) rather than unsupervised phase inference from expression alone
- Data are bulk rather than single-cell, or the target task is cell-cycle (not circadian)
  phase ordering, for which the method's ~20-gene core clock prior is inappropriate
- Only a single fixed pass of gene fitting is needed without iterative refinement
  against core clock Bayesian evidence
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Tempo
  role: Bayesian variational inference package implementing the alternating Step 1
    (posterior optimization) / Step 2 (de novo cycler detection) loop
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Data structure holding the single-cell expression matrix input to Tempo
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Numeric backend for the gene parameter tables handled during posterior fitting
  repo: https://github.com/bauerbach95/tempo
- name: pandas
  role: Tabular handling of gene/cell metadata during iterative fitting
  repo: https://github.com/bauerbach95/tempo
- name: scanpy>=1.6
  role: scRNA-seq preprocessing dependency required by the tempo conda environment
  repo: https://github.com/bauerbach95/tempo
- name: pytorch>=1.9.0
  role: Deep-learning backend supporting variational optimization in the tempo environment
  repo: https://github.com/bauerbach95/tempo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:d929f9611984bc91ac204c7ac96874ee5e8e45e2e310f7e011cf5687e672b2da
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# iterative-gene-parameter-fitting

## Summary

Tempo's two-step iterative variational inference loop that alternates KL-divergence-based optimization of the approximate joint posterior q(theta, beta) (cell phase estimation) with de novo cycler detection, terminating when core clock gene Bayesian evidence worsens or a maximum iteration count is exceeded.

## When to use

Use when estimating single-cell circadian phases unsupervised from scRNA-seq expression (e.g., anndata matrices) and unknown clock-controlled genes must be identified de novo while quantifying phase uncertainty.

## When NOT to use

- The analysis goal assumes sample collection timing proxies circadian phase (time-course designs) rather than unsupervised phase inference from expression alone
- Data are bulk rather than single-cell, or the target task is cell-cycle (not circadian) phase ordering, for which the method's ~20-gene core clock prior is inappropriate
- Only a single fixed pass of gene fitting is needed without iterative refinement against core clock Bayesian evidence

## Inputs

- Single-cell scRNA-seq expression matrix (anndata object)
- User-supplied core clock gene list
- Prior distributions over model parameters (theta, beta)

## Outputs

- Optimized approximate joint posterior q(theta, beta)
- Per-cell circadian phase estimates with uncertainty quantification
- De novo detected cycling genes (cyclers)

## How to apply

Initialize the Tempo model (Python >= 3.8 backend, numpy/pandas for numeric tables, anndata for the expression matrix) with user-supplied core clock genes and prior distributions. Execute Step 1: minimize the KL divergence between the approximate joint posterior q(theta, beta) and the true posterior to fit cell phases and cycler parameters jointly using the current set of cycling genes. Execute Step 2: fit gene parameters for non-cycling genes conditional on the current phase estimates, then detect de novo cyclers. Repeat Steps 1-2 until the core-clock gene Bayesian evidence worsens relative to the prior iteration or falls below a random baseline, then output the optimized approximate joint posterior. Validate by running run_test.py in the activated tempo conda environment; success is a completion under 5 minutes printing 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Bayesian variational inference package implementing the alternating Step 1 (posterior optimization) / Step 2 (de novo cycler detection) loop) — https://github.com/bauerbach95/tempo
- **anndata** (Data structure holding the single-cell expression matrix input to Tempo) — https://github.com/bauerbach95/tempo
- **numpy** (Numeric backend for the gene parameter tables handled during posterior fitting) — https://github.com/bauerbach95/tempo
- **pandas** (Tabular handling of gene/cell metadata during iterative fitting) — https://github.com/bauerbach95/tempo
- **scanpy>=1.6** (scRNA-seq preprocessing dependency required by the tempo conda environment) — https://github.com/bauerbach95/tempo
- **pytorch>=1.9.0** (Deep-learning backend supporting variational optimization in the tempo environment) — https://github.com/bauerbach95/tempo

## Examples

```
conda activate tempo; python run_test.py
```

## Evaluation signals

- run_test.py in the activated tempo conda environment completes in under 5 minutes and prints the message 'SUCCESSFULLY FINISHED'
- Iterative loop terminates only when core clock gene Bayesian evidence worsens relative to the prior iteration or falls below the evidence of a random baseline
- Output posterior q(theta, beta) yields well-calibrated phase uncertainty quantifications (article claims well-calibrated uncertainty)
- Phase estimates are more accurate than existing unsupervised methods on droplet-based scRNA-seq benchmarks

## Limitations

- Simulated analyses suggest de novo cycler detection issues may stem from assuming 24 h sinusoidal CCG expression means; more flexible waveform modeling is needed
- Subpopulation-specific gene amplitude variation is not explicitly modeled; mapping from a cell-state embedding to sinusoidal parameters would be an ideal solution
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, though less important for single-sample unsynchronized scRNA-seq
- Requires >=8GB RAM for in-memory operations; installation requires mini-conda and takes less than 5 minutes

## Evidence

- [other] Tempo repeats Step 1 (KL-divergence minimization of the approximate posterior using the current cycling genes) and Step 2 (de novo cycler detection) until the core clock gene Bayesian evidence worsens: [evidence span withheld — anchor status: not_found]
- [intro] Tempo, a Bayesian variational inference approach, for circadian phase inference: "we developed Tempo, a Bayesian variational inference approach, for circadian phase inference."
- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED".: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
