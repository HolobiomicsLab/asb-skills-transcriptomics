---
name: synthetic-network-simulation
description: Use when benchmarking multi-subject dynamic community detection (e.g.
  MuDCoD) and you need synthetic S x T x N adjacency tensors with known node labels
  across subjects and time points.
when_to_use_negative:
- You already have real subject-specific co-expression networks — instead use Dozer
  to filter genes and construct robust networks from scRNA-seq data
- You need to detect communities on existing networks rather than generate labeled
  benchmark data — run MuDCoD fit_predict on the input networks instead
- Your analysis involves only a single subject or a single time point, since this
  simulator targets the multi-subject, multi-time-point setting
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: MuSDynamicDCBM
  role: Simulation model whose instance generates multi-subject dynamic networks via
    simulate_mus_dynamic_dcbm(setting=1) for SSoT and setting=2 for SSoS scenarios
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Serializes the simulated S x T x 500 x 500 adjacency arrays and S x T x 500
    label arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as input
- name: MuDCoD
  role: Downstream consumer of the simulated networks and ground-truth labels; predicted
    communities (pred_MuDCoD) are compared to z_mus_dynamic_true to evaluate accuracy
  repo: https://github.com/bo1929/MuDCoD
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:52a8816965dd0c3eed6f2492c1ae52a8c1855b5c26734406dda1a72dd67897a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# synthetic-network-simulation

## Summary

Simulate multi-subject dynamic gene co-expression networks with ground-truth community labels using the MuSDynamicDCBM simulator, in SSoT and SSoS variants, to benchmark MuDCoD community detection.

## When to use

Use when benchmarking multi-subject dynamic community detection (e.g. MuDCoD) and you need synthetic S x T x N adjacency tensors with known node labels across subjects and time points.

## When NOT to use

- You already have real subject-specific co-expression networks — instead use Dozer to filter genes and construct robust networks from scRNA-seq data
- You need to detect communities on existing networks rather than generate labeled benchmark data — run MuDCoD fit_predict on the input networks instead
- Your analysis involves only a single subject or a single time point, since this simulator targets the multi-subject, multi-time-point setting

## Inputs

- Simulation parameters: node count n=500, module count k=10, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, random seed
- Scenario setting flag (setting=1 for SSoT, setting=2 for SSoS)
- Set of time horizons T in {2, 4, 8}

## Outputs

- adj_mus_dynamic.npy — S x T x 500 x 500 numpy adjacency array (S=16 subjects, T time points)
- z_mus_dynamic_true.npy — S x T x 500 numpy ground-truth community membership array

## How to apply

Construct a MuSDynamicDCBM simulation model instance with desired parameters (e.g. n=500, k=10, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0). Call simulate_mus_dynamic_dcbm(setting=1) for the SSoT scenario, where subjects are siblings at the initial time step then evolve independently, and setting=2 for the SSoS scenario, where subjects are siblings at each time point. Generate network series for each time horizon in {2, 4, 8}, yielding S x T x 500 x 500 adjacency arrays and matching S x T x 500 community membership arrays. Serialize the networks to adj_mus_dynamic.npy and labels to z_mus_dynamic_true.npy with numpy.save. Validate the layout: last two dimensions are 500 x 500 adjacency matrices, first dimension indexes subjects, second indexes time points, and both outputs share the 16 x T x 500 subject-time-node structure.

## Related tools

- **MuSDynamicDCBM** (Simulation model whose instance generates multi-subject dynamic networks via simulate_mus_dynamic_dcbm(setting=1) for SSoT and setting=2 for SSoS scenarios) — https://github.com/bo1929/MuDCoD
- **numpy** (Serializes the simulated S x T x 500 x 500 adjacency arrays and S x T x 500 label arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as input)
- **MuDCoD** (Downstream consumer of the simulated networks and ground-truth labels; predicted communities (pred_MuDCoD) are compared to z_mus_dynamic_true to evaluate accuracy) — https://github.com/bo1929/MuDCoD

## Examples

```
mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=2)  # SSoS: subjects are siblings at each time point
```

## Evaluation signals

- adjacency array's last two dimensions form 500 x 500 matrices
- Both outputs share the 16 x T x 500 subject-time-node layout (S=16, T in {2,4,8}, 500 nodes)
- z_mus_dynamic_true.npy last dimension is an array of 500 community labels
- Downstream: compare pred_MuDCoD against z_mus_dynamic_true (e.g. mean ARI per subject-time-point) to confirm labels are usable for evaluation

## Limitations

- No concrete numeric ARI results are reported in the article text (code prints computed values but no output values appear)
- Two scenario variants exist and must be chosen deliberately: setting=1 (SSoT — siblings only at the initial time step, then independent evolution) vs setting=2 (SSoS — siblings at each time point)
- Simulated networks lack the Dozer gene-filtering step used for real scRNA-seq-derived networks

## Evidence

- [methods] Initialize a MuSDynamicDCBM simulation model with SSoS/SSoT parameters: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] Run the simulator in both scenario variants: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
- [methods] numpy serialization of simulated arrays: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] Validate simulation by comparing predictions to true labels: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [readme] MuDCoD is installable and reusable as a Python package: "You can either clone the repository and crate a new virtual environment using poetry as described below, or simply use `pip install mudcod`."
