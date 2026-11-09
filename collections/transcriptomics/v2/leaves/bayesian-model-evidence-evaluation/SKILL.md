---
name: bayesian-model-evidence-evaluation
description: Use when an unsupervised variational inference loop (e.g., Tempo phase
  inference) needs a principled termination criterion, or when deciding whether adding
  de novo detected cycler genes improves or worsens model fit to core clock genes.
when_to_use_negative:
- Sample timing is trusted and a time-course design can proxy circadian phase without
  unsupervised inference from gene expression.
- The model is a single fixed pass and no iterative gene-set update (Step 2 de novo
  cycler detection) is performed, so there is no evidence trajectory to evaluate.
- Bayesian evidence for the current gene set cannot be computed because core clock
  genes or priors were never supplied.
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Bayesian variational inference framework whose iterative Step 1/Step 2 loop
    is gated by core-clock Bayesian evidence comparison
  repo: https://github.com/bauerbach95/tempo
- name: pytorch
  role: Backend supporting variational optimization of q(theta, beta) during each
    iteration
- name: anndata
  role: Container for the scRNA-seq expression matrix fed to the evidence-gated inference
    loop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:c0fa7d340282d30010d38595ad66ffc58abb595dd379118b104c03cda6821497
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# bayesian-model-evidence-evaluation

## Summary

Iterative Bayesian model comparison used in Tempo's variational inference loop: at each iteration, evaluate Bayesian evidence for user-supplied core clock genes against the prior iteration and a random baseline, and stop when evidence degrades. This criterion controls when the optimized approximate joint posterior q(theta, beta) is considered final.

## When to use

Use when an unsupervised variational inference loop (e.g., Tempo phase inference) needs a principled termination criterion, or when deciding whether adding de novo detected cycler genes improves or worsens model fit to core clock genes.

## When NOT to use

- Sample timing is trusted and a time-course design can proxy circadian phase without unsupervised inference from gene expression.
- The model is a single fixed pass and no iterative gene-set update (Step 2 de novo cycler detection) is performed, so there is no evidence trajectory to evaluate.
- Bayesian evidence for the current gene set cannot be computed because core clock genes or priors were never supplied.

## Inputs

- scRNA-seq expression matrix (anndata object) from droplet- or plate-based data
- user-supplied list of core circadian clock genes (~20 moderately expressed genes)
- prior distributions for model parameters theta and beta

## Outputs

- optimized approximate joint posterior q(theta, beta) over phases and cycler parameters
- per-iteration Bayesian evidence values for core clock genes (termination diagnostics)
- estimated circadian phase per cell with uncertainty quantification

## How to apply

Initialize the Tempo model from user-supplied core clock genes and prior distributions (Python >= 3.8 environment with numpy, pandas, anndata, scanpy>=1.6, pytorch>=1.9.0). Alternate Step 1 — minimizing the KL divergence between the approximate joint posterior q(theta, beta) and the true posterior to jointly fit cell phases and cycler parameters — with Step 2, fitting gene parameters for non-cycling genes conditional on the current phase estimates. After each iteration, compare the Bayesian evidence of the core clock genes against the previous iteration's evidence and against the evidence of a random baseline. Terminate when core-clock Bayesian evidence worsens relative to the prior iteration or falls below the random baseline; otherwise output the optimized approximate joint posterior q(theta, beta) as final. Validate the environment by running the packaged test which should print 'SUCCESSFULLY FINISHED' in under 5 minutes.

## Related tools

- **Tempo** (Bayesian variational inference framework whose iterative Step 1/Step 2 loop is gated by core-clock Bayesian evidence comparison) — https://github.com/bauerbach95/tempo
- **pytorch** (Backend supporting variational optimization of q(theta, beta) during each iteration)
- **anndata** (Container for the scRNA-seq expression matrix fed to the evidence-gated inference loop)

## Examples

```
conda activate tempo; python run_test.py
```

## Evaluation signals

- Core-clock Bayesian evidence improves or stays stable across iterations; termination occurs only when evidence worsens versus the prior iteration or drops below the random baseline.
- The final output is the optimized approximate joint posterior q(theta, beta), not an intermediate iterate.
- Phase estimates carry well-calibrated uncertainty quantifications (per the paper's abstract claim).
- Validation run: python run_test.py in the activated tempo conda environment completes in under 5 minutes and prints 'SUCCESSFULLY FINISHED'.

## Limitations

- Simulated analyses suggest de novo cycler detection issues may stem from assuming 24 h sinusoidal CCG expression means; more flexible waveform modeling is needed.
- Technical effects are not explicitly modeled, which becomes necessary for data collected as multiple samples over time.
- Subpopulation-specific gene amplitude variation is not explicitly modeled.
- Requires >=8GB RAM for most datasets and Python >= 3.8 with pytorch>=1.9.0; tested on macOS 10.14.5/12.2.1 and CentOS Linux 7.

## Evidence

- [methods] Termination via Bayesian evidence: [evidence span withheld — anchor status: not_found]
- [intro] Objective of the variational loop: "we developed Tempo, a Bayesian variational inference approach, for circadian phase inference."
- [abstract] Uncertainty calibration as output quality: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods and provides well-calibrated uncertainty quantifications"
- [readme] Validation procedure from README: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [discussion] Sinusoidal assumption limitation: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
