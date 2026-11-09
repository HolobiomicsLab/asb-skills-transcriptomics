---
name: temporal-and-subject-heterogeneity-parameterization
description: Use when benchmarking multi-subject dynamic community detection (e.g.,
  MuDCoD) and you need simulated S x T x N gene co-expression networks with controlled
  similarity between adjacent time points and between subjects, plus ground-truth
  community labels.
when_to_use_negative:
- Input is real scRNA-seq data rather than a simulation target — use Dozer to filter
  genes and construct robust networks instead of the DCBM simulator.
- Analysis involves only a single subject or a single time point — the SSoS/SSoT multi-subject,
  multi-time simulation layout (S x T x N) does not apply.
- You need community predictions rather than benchmark data with known labels — use
  MuDCoD fit_predict, not the MuSDynamicDCBM simulator.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MuSDynamicDCBM
  role: Simulation model class instantiated with heterogeneity parameters (r_time,
    r_subject) to generate multi-subject dynamic DCBM networks; its simulate_mus_dynamic_dcbm(setting=1|2)
    method produces the S x T x N x N adjacency arrays and true memberships
  repo: https://github.com/bo1929/MuDCoD
- name: MuDCoD
  role: 'Downstream benchmark target: the simulated networks with ground-truth communities
    are used to evaluate MuDCoD''s smoothed community detection'
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Serializes the S x T x 500 x 500 network arrays and S x T x 500 membership
    arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as network input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:404333cc7ca42eca3e513f59e782364165e52779be5e232a9d64aeccb4d3e58b
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# temporal-and-subject-heterogeneity-parameterization

## Summary

Parameterizes the MuSDynamicDCBM multi-subject dynamic network simulator to control temporal progression (r_time) and subject heterogeneity (r_subject), using SSoT (siblings at initial time, then independent evolution) or SSoS (siblings at each time point) settings to generate benchmark networks with ground-truth communities for MuDCoD evaluation.

## When to use

Use when benchmarking multi-subject dynamic community detection (e.g., MuDCoD) and you need simulated S x T x N gene co-expression networks with controlled similarity between adjacent time points and between subjects, plus ground-truth community labels.

## When NOT to use

- Input is real scRNA-seq data rather than a simulation target — use Dozer to filter genes and construct robust networks instead of the DCBM simulator.
- Analysis involves only a single subject or a single time point — the SSoS/SSoT multi-subject, multi-time simulation layout (S x T x N) does not apply.
- You need community predictions rather than benchmark data with known labels — use MuDCoD fit_predict, not the MuSDynamicDCBM simulator.

## Inputs

- Simulation parameters: n=500 nodes, k=10 modules, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed
- Time horizons {2, 4, 8} as the number of time points T
- Scenario selector: setting=1 (SSoT) or setting=2 (SSoS) passed to simulate_mus_dynamic_dcbm

## Outputs

- S x T x 500 x 500 adjacency array (subjects x time points x nodes x nodes) serialized to adj_mus_dynamic.npy via numpy.save
- S x T x 500 ground-truth community membership array (500 labels per subject-time) serialized to z_mus_dynamic_true.npy

## How to apply

Initialize a MuSDynamicDCBM simulation model instance with desired parameters (e.g., n=500 nodes, k=10 modules, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0). Choose the heterogeneity scenario via the simulate_mus_dynamic_dcbm setting argument: setting=1 yields the SSoT scenario where subjects are siblings at the initial time step then evolve independently, while setting=2 yields the SSoS scenario where gene modules of the ancestor network progress over time with controllable similarity between adjacent time points and each subject's modules are realizations from that ancestor. Generate network series for each time horizon in {2, 4, 8} and collect the resulting S x T x 500 x 500 adjacency arrays. Serialize networks with numpy.save to adj_mus_dynamic.npy and the corresponding S x T x 500 ground-truth memberships to z_mus_dynamic_true.npy. The rationale is that r_time controls similarity between adjacent time points and r_subject controls sharing among subject networks, so varying these (and the setting) probes MuDCoD's eigenvector smoothing across both axes of heterogeneity.

## Related tools

- **MuSDynamicDCBM** (Simulation model class instantiated with heterogeneity parameters (r_time, r_subject) to generate multi-subject dynamic DCBM networks; its simulate_mus_dynamic_dcbm(setting=1|2) method produces the S x T x N x N adjacency arrays and true memberships) — https://github.com/bo1929/MuDCoD
- **MuDCoD** (Downstream benchmark target: the simulated networks with ground-truth communities are used to evaluate MuDCoD's smoothed community detection) — https://github.com/bo1929/MuDCoD
- **numpy** (Serializes the S x T x 500 x 500 network arrays and S x T x 500 membership arrays to .npy files; MuDCoD takes multi-dimensional numpy arrays as network input)

## Examples

```
from mudcod.simulation import MuSDynamicDCBM; model = MuSDynamicDCBM(n=500, k=10, p_in=(0.2, 0.4), p_out=(0.05, 0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0); adj, z = model.simulate_mus_dynamic_dcbm(setting=2)
```

## Evaluation signals

- Network array's last two dimensions form 500 x 500 adjacency matrices.
- Both adj_mus_dynamic.npy and z_mus_dynamic_true.npy share the 16 x T x 500 subject-time-node layout, with networks additionally carrying the S x T x 500 x 500 shape.
- Community membership labels are an array of 500 labels in the last dimension of z_mus_dynamic_true.npy, enabling comparison against pred_MuDCoD for accuracy/ARI.
- Generated series exist for each requested time horizon in {2, 4, 8}, matching the chosen setting (1 = SSoT, 2 = SSoS).
- Re-running with the same seed=0 reproduces the same simulated networks (deterministic parameterization).

## Limitations

- No concrete numeric ARI results are reported in the article text (the code prints computed values but no output values appear), so expected accuracy ranges cannot be fixed in advance.
- The simulator is a synthetic DCBM benchmark; for real scRNA-seq networks the article suggests Dozer to filter genes and construct robust networks instead.
- Setting choice changes the null hypothesis being tested: SSoT has subjects sharing information only at the initial time step, so results under setting=1 do not transfer to the SSoS case.

## Evidence

- [methods] The MuS-Dynamic-DCBM simulator includes an SSoS mode in which gene modules of an ancestor network progress over time with controllable similarity between adjacent time points: [evidence span withheld — anchor status: not_found]
- [methods] First construct a MuSDynamicDCBM instance, i.e., simulation model, with desired parameters.: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] MuDCoD takes multi-dimensional `numpy` arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [abstract] MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
- [methods] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
