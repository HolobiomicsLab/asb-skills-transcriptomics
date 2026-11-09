---
name: degree-corrected-block-model-parameterization
description: Use when you need simulated multi-subject dynamic networks with known
  community labels to benchmark community detection — e.g. sweeping r_time, r_subject,
  and time_horizon T for MuDCoD evaluation.
when_to_use_negative:
- Input is already a real network tensor (e.g. Dozer-constructed scRNA-seq co-expression
  networks) — use MuDCoD fit_predict directly rather than the DCBM simulator
- You need a static single-time-point network benchmark with no subject dimension
  — a plain DCBM/spectral baseline suffices
- Ground-truth community labels already exist, so simulating new z_true memberships
  would be redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MuSDynamicDCBM
  role: Simulation model class whose instance parameters (n, k, p_in, p_out, num_subjects,
    r_time, r_subject) define the DCBM and whose simulate_mus_dynamic_dcbm(setting)
    call generates the multi-subject dynamic networks and true memberships
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Holds and serializes the multi-dimensional adjacency and membership arrays
    (numpy.save to adj_mus_dynamic.npy and z_mus_dynamic_true.npy); MuDCoD takes multi-dimensional
    numpy arrays as network input
  repo: https://github.com/bo1929/MuDCoD
- name: MuDCoD
  role: Downstream consumer of the simulated adjacency tensor; the DCBM ground-truth
    labels are compared against MuDCoD predictions to evaluate accuracy
  repo: https://github.com/bo1929/MuDCoD
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:a793163f7374c45de0f79b77b2a6f103a418571b9bdee2786f4f98cc6a86630c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Degree-corrected block model parameterization

## Summary

Parameterizes a multi-subject dynamic degree-corrected block model (MuSDynamicDCBM) simulator to generate synthetic gene co-expression networks with ground-truth community labels, providing controlled benchmark inputs (S x T x n adjacency tensors plus true memberships) for evaluating MuDCoD against baseline community detection methods.

## When to use

Use when you need simulated multi-subject dynamic networks with known community labels to benchmark community detection — e.g. sweeping r_time, r_subject, and time_horizon T for MuDCoD evaluation.

## When NOT to use

- Input is already a real network tensor (e.g. Dozer-constructed scRNA-seq co-expression networks) — use MuDCoD fit_predict directly rather than the DCBM simulator
- You need a static single-time-point network benchmark with no subject dimension — a plain DCBM/spectral baseline suffices
- Ground-truth community labels already exist, so simulating new z_true memberships would be redundant

## Inputs

- Simulation parameter set (n, k, p_in range, p_out range, num_subjects, r_time, r_subject, time_horizon T, seed)
- Scenario choice: setting=1 (SSoT) or setting=2 (SSoS)

## Outputs

- S x T x 500 x 500 numpy adjacency array (first dimension subjects, second time points) saved as adj_mus_dynamic.npy
- S x T x 500 ground-truth community membership array (last dimension 500 labels) saved as z_mus_dynamic_true.npy

## How to apply

Instantiate a MuSDynamicDCBM simulation model with the paper's benchmark parameters: n=500 nodes, k=10 communities, p_in=(0.2, 0.4) for within-module edge probabilities, p_out=(0.05, 0.1) for between-module probabilities, num_subjects=16, r_time=0.4, r_subject=0.2, and a fixed seed. Call simulate_mus_dynamic_dcbm(setting=1) for the SSoT scenario (subjects are siblings at the initial time step then evolve independently) or setting=2 for the SSoS scenario (subjects are siblings at each time point), sweeping time_horizon T in {2, 4, 8} and drawing 100 replicates per configuration for benchmarking. Collect the resulting S x T x 500 x 500 adjacency arrays and the corresponding S x T x 500 ground-truth membership arrays, serializing them with numpy.save as adj_mus_dynamic.npy and z_mus_dynamic_true.npy. The r_time parameter controls similarity between adjacent time points of the ancestor network, and r_subject controls how much each subject's modules deviate from that progressing ancestor, so these are the levers for testing information sharing across the subject and time dimensions.

## Related tools

- **MuSDynamicDCBM** (Simulation model class whose instance parameters (n, k, p_in, p_out, num_subjects, r_time, r_subject) define the DCBM and whose simulate_mus_dynamic_dcbm(setting) call generates the multi-subject dynamic networks and true memberships) — https://github.com/bo1929/MuDCoD
- **numpy** (Holds and serializes the multi-dimensional adjacency and membership arrays (numpy.save to adj_mus_dynamic.npy and z_mus_dynamic_true.npy); MuDCoD takes multi-dimensional numpy arrays as network input) — https://github.com/bo1929/MuDCoD
- **MuDCoD** (Downstream consumer of the simulated adjacency tensor; the DCBM ground-truth labels are compared against MuDCoD predictions to evaluate accuracy) — https://github.com/bo1929/MuDCoD

## Examples

```
from mudcod.dcbm import MuSDynamicDCBM; model = MuSDynamicDCBM(n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0); A, z = model.simulate_mus_dynamic_dcbm(setting=1, time_horizon=8)
```

## Evaluation signals

- Network array's last two dimensions must form 500 x 500 adjacency matrices
- Both outputs share the 16 x T x 500 subject-time-node layout (S x T x n)
- adj_mus_dynamic.npy and z_mus_dynamic_true.npy load via numpy.load without error and have consistent S and T dimensions
- Predicted MuDCoD assignments (16 x 8 x 500 for T=8) can be compared against z_mus_dynamic_true using ARI to confirm communities are recoverable
- Convergence monitoring of downstream MuDCoD fit_predict terminated without error on the simulated tensor

## Limitations

- No concrete numeric ARI results are printed in the article text, so simulation validity must be checked via ARI computations rather than reported reference values
- The simulator's controllable similarity between adjacent time points (r_time) and subjects (r_subject) means outputs are scenario-dependent; setting=1 and setting=2 produce structurally different sibling relationships and are not interchangeable
- Simulation-based benchmarks do not capture real scRNA-seq noise; the article suggests Dozer filtering for real data networks rather than relying on DCBM realism

## Evidence

- [methods] First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters.: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] numpy input format: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] simulation call: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
- [other] SSoS ancestor progression: [evidence span withheld — anchor status: not_found]
- [intro] smoothing rationale: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] evaluation comparison: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
