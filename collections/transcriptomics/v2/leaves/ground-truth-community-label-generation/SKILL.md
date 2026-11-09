---
name: ground-truth-community-label-generation
description: Use when benchmarking MuDCoD (or similar multi-subject dynamic community
  detection) and you need simulated S x T x N adjacency tensors paired with ground-truth
  node community labels.
when_to_use_negative:
- Input is real scRNA-seq data rather than simulated networks — use Dozer to filter
  genes and construct robust networks instead of MuSDynamicDCBM
- You need a single-subject or static (non-dynamic, non-multi-subject) network simulation;
  MuSDynamicDCBM specifically produces S x T subject-time network series
- Ground-truth labels already exist for the networks being analyzed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuSDynamicDCBM
  role: Simulation model class instantiated with desired parameters to generate multi-subject
    dynamic networks with ground-truth community labels (SSoS and SSoT variants)
  repo: https://github.com/bo1929/MuDCoD
- name: MuDCoD
  role: 'Downstream consumer: its fit_predict community predictions are compared against
    the generated ground-truth labels to evaluate accuracy'
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Serializes the S x T x 500 x 500 adjacency arrays and S x T x 500 membership
    arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:dd9816911c94fbcc05d77f4bb6dedc28179a2f7c9dd73e8832961e73f6ec7634
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# ground-truth-community-label-generation

## Summary

Generates multi-subject dynamic gene co-expression networks with known ground-truth community labels using the MuSDynamicDCBM simulator, so that MuDCoD community predictions can be benchmarked against the true memberships.

## When to use

Use when benchmarking MuDCoD (or similar multi-subject dynamic community detection) and you need simulated S x T x N adjacency tensors paired with ground-truth node community labels.

## When NOT to use

- Input is real scRNA-seq data rather than simulated networks — use Dozer to filter genes and construct robust networks instead of MuSDynamicDCBM
- You need a single-subject or static (non-dynamic, non-multi-subject) network simulation; MuSDynamicDCBM specifically produces S x T subject-time network series
- Ground-truth labels already exist for the networks being analyzed

## Inputs

- Simulation parameters: n=500 nodes, k=10 modules, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0
- Time horizons {2, 4, 8}
- Scenario choice: setting=1 (SSoT) or setting=2 (SSoS)

## Outputs

- S x T x 500 x 500 adjacency array serialized to adj_mus_dynamic.npy via numpy.save (subjects x time points x nodes x nodes)
- S x T x 500 ground-truth community membership array serialized to z_mus_dynamic_true.npy (last dimension an array of 500 labels)

## How to apply

Instantiate a MuSDynamicDCBM simulation model with the desired parameters (e.g. n=500, k=10, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0). Call simulate_mus_dynamic_dcbm(setting=1) for the SSoT scenario, where subjects are siblings at the initial time step and then evolve independently, and setting=2 for the SSoS scenario, where subjects are siblings at each time point; in SSoS mode, gene modules of an ancestor network progress over time with controllable similarity between adjacent time points, and each subject's modules at time t are realizations from this progressing ancestor. Generate series for each time horizon in {2, 4, 8}, collect the resulting S x T x 500 x 500 adjacency arrays (first dimension indexes subjects, second time points), and serialize them with numpy.save to adj_mus_dynamic.npy. Serialize the corresponding S x T x 500 ground-truth community membership arrays to z_mus_dynamic_true.npy. Later, compare predicted communities (pred_MuDCoD) against z_mus_dynamic_true to evaluate accuracy, e.g. via ARI.

## Related tools

- **MuSDynamicDCBM** (Simulation model class instantiated with desired parameters to generate multi-subject dynamic networks with ground-truth community labels (SSoS and SSoT variants)) — https://github.com/bo1929/MuDCoD
- **MuDCoD** (Downstream consumer: its fit_predict community predictions are compared against the generated ground-truth labels to evaluate accuracy) — https://github.com/bo1929/MuDCoD
- **numpy** (Serializes the S x T x 500 x 500 adjacency arrays and S x T x 500 membership arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as input)

## Examples

```
from mudcod.simulation import MuSDynamicDCBM; sim = MuSDynamicDCBM(n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0); adj, z = sim.simulate_mus_dynamic_dcbm(setting=2)
```

## Evaluation signals

- The network array's last two dimensions form 500 x 500 adjacency matrices
- Both adj_mus_dynamic.npy and z_mus_dynamic_true.npy share the 16 x T x 500 subject-time-node layout (with T in {2, 4, 8})
- The membership array's last dimension is an array of 500 community labels, one per node
- Adjacency arrays contain exactly S x T x 500 x 500 values, with first dimension indexing subjects and second indexing time points
- Comparing pred_MuDCoD against z_mus_dynamic_true yields a computable accuracy metric (e.g. mean ARI) rather than a shape mismatch

## Limitations

- The article reports no concrete numeric ARI results — code prints computed values (e.g. mean(ARI) of MuDCoD) but output values do not appear in the text
- Ground truth exists only for simulated data; for real scRNA-seq datasets (e.g. Jerber et al. 2021 dopaminergic neuron differentiation), networks must be constructed with Dozer and have no true labels
- The simulator requires a MuDCoD installation environment (Python 3.9, poetry/pip install mudcod)

## Evidence

- [methods] MuSDynamicDCBM simulation model instantiation: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] simulation call: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
- [methods] numpy input format: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] evaluation against true labels: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [readme] pip install: "You can either clone the repository and crate a new virtual environment using poetry as described below, or simply use `pip install mudcod`."
