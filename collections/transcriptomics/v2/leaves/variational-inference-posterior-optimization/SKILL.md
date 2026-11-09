---
name: variational-inference-posterior-optimization
description: Use when unsupervised circadian phase inference is needed on droplet-
  or plate-based scRNA-seq data, and the goal is an optimized approximate joint posterior
  q(theta, beta) with calibrated uncertainty rather than heuristic phase ordering.
when_to_use_negative:
- 'Data are not single-cell: bulk time-course designs where sample collection timing
  can proxy circadian phase do not need unsupervised phase inference'
- Only CCGs already known a priori and no phase estimates are needed — the de novo
  cycler detection loop is unnecessary overhead
- The transcript likelihood model must follow a non-sinusoidal 24 h waveform; Tempo
  assumes CCG expression means follow 24 h sinusoidal patterns, and the paper notes
  this may degrade de novo cycler detection
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Tempo
  role: Bayesian variational inference package whose two-step loop is optimized by
    this skill; installed in the 'tempo' conda environment via install.sh
  repo: https://github.com/bauerbach95/tempo
- name: Python >= 3.8
  role: Runtime backend required to execute the Tempo variational inference code
- name: numpy
  role: Numeric handling of expression tables during posterior optimization
- name: pandas
  role: Tabular data handling for gene/cell metadata in the inference loop
- name: anndata
  role: Container format for the scRNA-seq expression matrix input to Tempo
- name: scanpy>=1.6
  role: scRNA-seq preprocessing dependency of the Tempo environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:2d33b0404992be878d85ab3f9e70eed962d6e429aba3441120b87a5a609150bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Python >= 3.8
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# variational-inference-posterior-optimization

## Summary

Tempo's Bayesian variational inference procedure that iteratively optimizes an approximate joint posterior q(theta, beta) over cell circadian phases and cycler parameters, alternating posterior fitting with de novo cycler detection until Bayesian evidence for core clock genes stops improving.

## When to use

Use when unsupervised circadian phase inference is needed on droplet- or plate-based scRNA-seq data, and the goal is an optimized approximate joint posterior q(theta, beta) with calibrated uncertainty rather than heuristic phase ordering.

## When NOT to use

- Data are not single-cell: bulk time-course designs where sample collection timing can proxy circadian phase do not need unsupervised phase inference
- Only CCGs already known a priori and no phase estimates are needed — the de novo cycler detection loop is unnecessary overhead
- The transcript likelihood model must follow a non-sinusoidal 24 h waveform; Tempo assumes CCG expression means follow 24 h sinusoidal patterns, and the paper notes this may degrade de novo cycler detection

## Inputs

- Single-cell RNA-seq expression matrix as an anndata object (droplet- or plate-based scRNA-seq)
- User-supplied core circadian clock gene set (~20 moderately expressed genes)
- Prior distributions over model parameters (theta, beta)

## Outputs

- Optimized approximate joint posterior q(theta, beta) over cell phases and cycler parameters
- Per-cell circadian phase point estimates with uncertainty quantifications
- De novo identified cycling genes (CCGs)

## How to apply

Initialize the Tempo model (Python >= 3.8 backend with numpy, pandas, anndata, scanpy>=1.6, pytorch>=1.9.0) from user-supplied core clock genes and prior distributions. Execute Step 1: minimize the KL divergence between the approximate joint posterior q(theta, beta) and the true posterior to jointly fit cell phases and cycler parameters using the current set of cycling genes. Execute Step 2: fit gene parameters for de novo cycler candidates conditional on the current phase estimates. Repeat the two steps until the core clock gene Bayesian evidence worsens relative to the prior iteration or falls below the evidence of a random baseline, or the maximum number of iterations is exceeded; output the optimized approximate joint posterior q(theta, beta). Validate the environment with the packaged test, which must finish in under 5 minutes and print 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Bayesian variational inference package whose two-step loop is optimized by this skill; installed in the 'tempo' conda environment via install.sh) — https://github.com/bauerbach95/tempo
- **Python >= 3.8** (Runtime backend required to execute the Tempo variational inference code)
- **numpy** (Numeric handling of expression tables during posterior optimization)
- **pandas** (Tabular data handling for gene/cell metadata in the inference loop)
- **anndata** (Container format for the scRNA-seq expression matrix input to Tempo)
- **scanpy>=1.6** (scRNA-seq preprocessing dependency of the Tempo environment)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- Run 'python run_test.py' in the activated tempo conda environment; it must complete in under 5 minutes and print 'SUCCESSFULLY FINISHED'
- The iterative loop terminates only when core clock gene Bayesian evidence worsens relative to the prior iteration or falls below a random baseline — confirm the stopping criterion, not the iteration count, ends the run
- The final returned object is the optimized approximate joint posterior q(theta, beta), not merely per-cell phase assignments
- Uncertainty quantifications accompanying phase estimates should be well-calibrated, as demonstrated for Tempo
- Phase estimates should be more accurate than existing unsupervised phase inference methods on the same droplet-based data

## Limitations

- De novo cycler detection may underperform because Tempo assumes CCG expression means follow 24 h sinusoidal patterns across the circadian cycle; more flexible waveform modeling is needed
- Technical effects are not explicitly modeled, which becomes more necessary for data collected as multiple samples over time
- Subpopulation-specific gene amplitude variation is not modeled; a mapping from a low-dimensional cell-state embedding to sinusoidal parameters is suggested as a more ideal solution
- Prior unsupervised methods' transcript likelihood choices poorly approximate droplet-based generative distributions — mismatch between likelihood and platform is a key failure mode the variational objective must address

## Evidence

- [abstract] we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [methods] Tempo requires python >= 3.8: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] activation and validation commands: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [discussion] sinusoidal waveform assumption: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [intro] poor droplet likelihood approximation motivates the variational model: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution"
