---
name: simulation-parameter-sweep-design
description: Use when benchmarking community detection (or similar inference) methods
  on simulated multi-subject dynamic networks and you need to isolate the effect of
  information-sharing levels (r_subject, r_time) and time horizon T on accuracy.
when_to_use_negative:
- The goal is applied community detection on real scRNA-seq-derived co-expression
  networks rather than controlled benchmarking — then use Dozer-filtered networks
  directly instead of a synthetic sweep.
- The simulation axes of interest are not information-sharing levels or time horizon;
  other MuSDynamicDCBM parameters (n, k, p_in/p_out) should be fixed, not swept, if
  the claim under test concerns r_time/r_subject/T.
- Only a single network configuration is needed (no comparison across parameter levels)
  — a one-shot MuSDynamicDCBM simulation suffices rather than a full sweep.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MuSDynamicDCBM
  role: Simulation model instantiated with desired parameters and used to generate
    multi-subject dynamic networks per sweep configuration via simulate_mus_dynamic_dcbm
  repo: https://github.com/bo1929/MuDCoD
- name: MuDCoD
  role: Method under evaluation whose smoothed spectral representations and predicted
    communities are scored by ARI at each sweep point (installed via `pip install
    mudcod`)
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Provides the multi-dimensional array format for network inputs and the hyperparameter
    arrays (alpha, beta) passed to MuDCoD fit_predict
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:3b08465f13a7c81dda671750c1aae55dd5411812c292a93e7d8d5d8cde73c4b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Simulation parameter sweep design

## Summary

Designs and executes controlled parameter sweeps over a simulated multi-subject dynamic network benchmark (MuSDynamicDCBM), varying r_subject, r_time, and time horizon T with fixed graph topology parameters and ~100 replicates per configuration, to measure how a detection method's accuracy (ARI) responds to each parameter.

## When to use

Use when benchmarking community detection (or similar inference) methods on simulated multi-subject dynamic networks and you need to isolate the effect of information-sharing levels (r_subject, r_time) and time horizon T on accuracy.

## When NOT to use

- The goal is applied community detection on real scRNA-seq-derived co-expression networks rather than controlled benchmarking — then use Dozer-filtered networks directly instead of a synthetic sweep.
- The simulation axes of interest are not information-sharing levels or time horizon; other MuSDynamicDCBM parameters (n, k, p_in/p_out) should be fixed, not swept, if the claim under test concerns r_time/r_subject/T.
- Only a single network configuration is needed (no comparison across parameter levels) — a one-shot MuSDynamicDCBM simulation suffices rather than a full sweep.

## Inputs

- MuSDynamicDCBM simulation configuration: n=500, k=10, num_subjects=16, p_in=(0.2,0.4), p_out=(0.05,0.1)
- Sweep axes: r_time levels, r_subject levels, time_horizon T in {2,4,8}, setting in {1 (SSoT), 2 (SSoS)}
- Replicate count per configuration (100 replicates)
- Method configurations to hold constant (e.g. MuDCoD alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30)

## Outputs

- Simulated adjacency tensors per replicate with ground-truth labels z_mus_dynamic_true
- Per-subject, per-time-point adjusted Rand index values for each method and configuration
- Aggregated mean-ARI table (ari_results.csv) across subjects, time points, and 100 replicates
- Fig. 3-style ARI-vs-parameter curves for the SSoS and SSoT settings

## How to apply

Construct a MuSDynamicDCBM instance with fixed structural parameters (n=500, k=10, num_subjects=16, p_in=(0.2,0.4), p_out=(0.05,0.1)) and sweep the axes of interest: r_time and r_subject levels, and T in {2,4,8}, drawing 100 replicates per configuration via simulate_mus_dynamic_dcbm under both setting=1 (SSoT) and setting=2 (SSoS) regimes. Hold the non-swept parameters constant across configurations so observed ARI differences are attributable to the swept parameter, and keep the ground-truth labels z_mus_dynamic_true from each replicate for downstream ARI evaluation. Choose sweep points that span the extremes where methods are expected to separate — e.g. r_time=0.0 (modules fully conserved across time) where MuDCoD shows a >20% mean-ARI gain over PisCES at T=8, and r_subject or r_time=0.5 where the Betzel-2019 baseline degrades to mean ARI <0.2. Fit every method (MuDCoD with alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, monitor_convergence=True, and the PisCES / Betzel-2019 / multi-subject baselines under identical configurations) on the same simulated adjacency tensors per replicate. Finally aggregate mean adjusted Rand index across subjects, time points, and replicates for each method-by-configuration cell, writing the aggregated table (ari_results.csv) and ARI-vs-parameter curves comparable to Fig. 3.

## Related tools

- **MuSDynamicDCBM** (Simulation model instantiated with desired parameters and used to generate multi-subject dynamic networks per sweep configuration via simulate_mus_dynamic_dcbm) — https://github.com/bo1929/MuDCoD
- **MuDCoD** (Method under evaluation whose smoothed spectral representations and predicted communities are scored by ARI at each sweep point (installed via `pip install mudcod`)) — https://github.com/bo1929/MuDCoD
- **numpy** (Provides the multi-dimensional array format for network inputs and the hyperparameter arrays (alpha, beta) passed to MuDCoD fit_predict)

## Examples

```
from mudcod.simulation import mus_dynamic_dcbm; A, z_true = mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)
```

## Evaluation signals

- At SSoT with r_time=0 and T=8, reproduced mean-ARI curves show MuDCoD exceeding PisCES by more than 20% (Fig. 3 pattern)
- Betzel-2019 baseline yields mean ARI <0.2 at r_subject=0.5 and/or r_time=0.5 in the reproduced curves
- Each configuration cell aggregates over exactly 100 replicates with mean ARI taken across subjects and time points
- Predicted community arrays are compared against z_mus_dynamic_true labels per subject and time point (ARI computed against the true simulation labels, not inferred ones)

## Limitations

- No concrete numeric ARI values appear in the paper text itself — evaluation must rely on reproducing the reported Fig. 3 qualitative patterns rather than exact printed numbers
- Method hyperparameters alpha and beta require cross-validation (examples/cross_validation.py); fixed values like 0.05 are choices, not validated optima
- Simulated ground truth comes only from the MuSDynamicDCBM generative model, so conclusions may not transfer to networks constructed from real scRNA-seq data (Dozer-filtered gene co-expression networks)

## Evidence

- [methods] MuSDynamicDCBM instance with desired parameters: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] simulate_mus_dynamic_dcbm invocation: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
- [methods] ARI evaluation against true labels: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [results] ARI gain finding at rtime=0, T=8: [evidence span withheld — anchor status: not_found]
- [readme] pip install path for reproducing simulation experiments: "You can either clone the repository and crate a new virtual environment using poetry as described below, or simply use `pip install mudcod`."
