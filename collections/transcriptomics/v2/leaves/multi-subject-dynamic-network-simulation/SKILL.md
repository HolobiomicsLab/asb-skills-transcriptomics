---
name: multi-subject-dynamic-network-simulation
description: Use when benchmarking multi-subject dynamic community detection (e.g.,
  MuDCoD vs PisCES/static spectral clustering/Betzel-2019) or when ground-truth subject
  x time x node labels are needed for ARI evaluation on simulated S x T x n x n adjacency
  tensors.
when_to_use_negative:
- Your data are real single-cell RNA-seq expression matrices rather than simulated
  networks — use Dozer to filter genes and construct personalized co-expression networks
  instead
- You need community detection on a single network with no subject or time dimension
  — the multi-subject dynamic simulator's S x T layout does not apply
- You lack ground-truth label requirements and only need clustering of observed networks
  without benchmarking
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MuSDynamicDCBM
  role: Simulation model class that generates multi-subject dynamic networks with
    ground-truth communities via simulate_mus_dynamic_dcbm in SSoT (setting=1) and
    SSoS (setting=2) modes
  repo: https://github.com/bo1929/MuDCoD
- name: MuDCoD
  role: Downstream consumer of simulated tensors; fit_predict produces smoothed spectral
    representations and predicted communities to compare against z_mus_dynamic_true
    via ARI
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Provides the multi-dimensional array format MuDCoD takes as network input
    and numpy.save serialization of adjacency and label arrays
- name: Dozer
  role: 'Recommended upstream tool for real scRNA-seq pipelines: filters genes and
    constructs robust personalized co-expression networks (not used for simulation
    itself)'
  repo: https://github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:5cd86490a583cd4f8e5da2f7fd879c8409551a65d3105deff4b57a8a696827df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multi-subject-dynamic-network-simulation

## Summary

Simulate multi-subject dynamic gene co-expression networks with ground-truth communities using the MuSDynamicDCBM model, in SSoT and SSoS variants, to benchmark MuDCoD against baseline community-detection methods.

## When to use

Use when benchmarking multi-subject dynamic community detection (e.g., MuDCoD vs PisCES/static spectral clustering/Betzel-2019) or when ground-truth subject x time x node labels are needed for ARI evaluation on simulated S x T x n x n adjacency tensors.

## When NOT to use

- Your data are real single-cell RNA-seq expression matrices rather than simulated networks — use Dozer to filter genes and construct personalized co-expression networks instead
- You need community detection on a single network with no subject or time dimension — the multi-subject dynamic simulator's S x T layout does not apply
- You lack ground-truth label requirements and only need clustering of observed networks without benchmarking

## Inputs

- MuSDynamicDCBM parameter configuration: n=500 nodes, k=10 modules, p_in=(0.2,0.4), p_out=(0.05,0.1), num_subjects=16
- Setting choice: setting=1 (SSoT) or setting=2 (SSoS)
- r_time and r_subject similarity levels and time_horizon T in {2,4,8}
- Optional seed for reproducibility (e.g., seed=0)

## Outputs

- S x T x 500 x 500 multi-subject dynamic adjacency array (serializable to adj_mus_dynamic.npy)
- S x T x 500 ground-truth community membership array z_mus_dynamic_true.npy
- Aggregated mean ARI tables (e.g., ari_results.csv) across subjects, time points, and replicates

## How to apply

Construct a MuSDynamicDCBM simulation instance with n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1), num_subjects=16, and chosen r_time/r_subject similarity levels (e.g., r_time=0.4, r_subject=0.2, seed=0). Call simulate_mus_dynamic_dcbm(setting=1) for SSoT (subjects are siblings at the initial time step then evolve independently) and setting=2 for SSoS (subjects are siblings at each time point), sweeping time_horizon T in {2,4,8} with ~100 replicates per configuration. Serialize the resulting S x T x 500 x 500 adjacency array (numpy.save to adj_mus_dynamic.npy) and the S x T x 500 true membership array (z_mus_dynamic_true.npy); the first dimension indexes subjects, the second time points. Downstream, fit MuDCoD with alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, monitor_convergence=True, then score predicted communities against z_mus_dynamic_true with adjusted Rand index per subject and time point.

## Related tools

- **MuSDynamicDCBM** (Simulation model class that generates multi-subject dynamic networks with ground-truth communities via simulate_mus_dynamic_dcbm in SSoT (setting=1) and SSoS (setting=2) modes) — https://github.com/bo1929/MuDCoD
- **MuDCoD** (Downstream consumer of simulated tensors; fit_predict produces smoothed spectral representations and predicted communities to compare against z_mus_dynamic_true via ARI) — https://github.com/bo1929/MuDCoD
- **numpy** (Provides the multi-dimensional array format MuDCoD takes as network input and numpy.save serialization of adjacency and label arrays)
- **Dozer** (Recommended upstream tool for real scRNA-seq pipelines: filters genes and constructs robust personalized co-expression networks (not used for simulation itself)) — https://github.com/shanlu01/Dozer

## Examples

```
from mudcod.mus_dynamic_dcbm import MuSDynamicDCBM; model = MuSDynamicDCBM(n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1), num_subjects=16, r_time=0.4, r_subject=0.2, seed=0); adj, z_true = model.simulate_mus_dynamic_dcbm(setting=2)
```

## Evaluation signals

- Adjacency array's last two dimensions are 500 x 500 matrices, and both outputs share the 16 x T x 500 subject-time-node layout
- z_mus_dynamic_true has shape S x T x 500 with integer community labels in the last dimension (k=10 modules)
- Downstream benchmark reproduces Fig. 3 patterns: MuDCoD achieves >20% mean-ARI gain over PisCES at SSoT with r_time=0 and T=8
- Betzel-2019 baseline yields mean ARI <0.2 at r_subject=0.5 and/or r_time=0.5
- Predicted communities (pred_MuDCoD) scored per subject/time point against z_mus_dynamic_true give finite ARI values aggregatable over 100 replicates

## Limitations

- The article text does not report concrete numeric ARI values (code prints computed values but no output values appear in the text), so expected ARI levels must be inferred from figures/claims
- Hyper-parameters alpha and beta require tuning; the authors suggest cross-validation (examples/cross_validation.py) rather than fixed values
- Simulator covers controlled similarity regimes (r_time, r_subject); behavior on real heterogeneous subject groupings (e.g., case vs control groups) requires extending the beta smoothing term

## Evidence

- [methods] SSoS mode with progressing ancestor networks: [evidence span withheld — anchor status: not_found]
- [methods] simulation instance parameters: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [methods] input array format: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] evaluation against truth: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [readme] hyper-parameter tuning advice: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] setting dispatch: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
