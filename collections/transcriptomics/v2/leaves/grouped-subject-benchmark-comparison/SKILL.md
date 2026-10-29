---
name: grouped-subject-benchmark-comparison
description: Use when you need to quantify whether a modification to MuDCoD's subject-smoothing
  term (e.g., replacing the global all-subject mean with group-specific means) improves
  community recovery, measured by ARI against known true labels on simulated networks.
when_to_use_negative:
- There are no known true community labels (e.g., real scRNA-seq networks without
  ground truth), since ARI against z_mus_dynamic_true is the sole evaluation signal
- Subjects have no predefined groupings — the standard MuDCoD β-penalty with global
  all-subject mean smoothing is already the appropriate model
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MuDCoD
  role: Baseline algorithm run via fit_predict; its β-penalty (pulling each subject's
    smoothed projection matrix toward the mean projection over all other subjects)
    is the component replaced by group-specific smoothing
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model used to generate grouped multi-subject dynamic networks with
    known true labels z_mus_dynamic_true for benchmarking
- name: numpy
  role: Constructs hyper-parameter arrays (alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S))
    and provides the multi-dimensional array input format for MuDCoD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:a9a8179dea4445dd300c42d856423490a0f5dcd1971253f42537b478705a6cbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# grouped-subject-benchmark-comparison

## Summary

Benchmarking a modified MuDCoD variant against the standard algorithm on simulated multi-subject dynamic networks by comparing adjusted Rand index (ARI) per subject and time point. This isolates the effect of replacing the global β-penalty (all-subject mean smoothing) with group-specific smoothing for case/control-style subject groupings.

## When to use

Use when you need to quantify whether a modification to MuDCoD's subject-smoothing term (e.g., replacing the global all-subject mean with group-specific means) improves community recovery, measured by ARI against known true labels on simulated networks.

## When NOT to use

- There are no known true community labels (e.g., real scRNA-seq networks without ground truth), since ARI against z_mus_dynamic_true is the sole evaluation signal
- Subjects have no predefined groupings — the standard MuDCoD β-penalty with global all-subject mean smoothing is already the appropriate model

## Inputs

- Simulated multi-subject dynamic network adjacency array of shape S x T x n x n (S=16 subjects, T=8 time points, n=500 nodes) from MuSDynamicDCBM
- True community labels z_mus_dynamic_true (16 x 8 x 500 array) from the simulation model
- Predefined subject group assignments (e.g., case vs control) for the 16 subjects

## Outputs

- ari_comparison.csv containing per-subject/per-time-point and mean adjusted Rand index values for standard MuDCoD and the group-aware variant
- Predicted community assignments pred_MuDCoD for each variant (16 x 8 x 500 arrays)

## How to apply

Simulate grouped multi-subject dynamic networks with MuSDynamicDCBM (n=500 nodes, k=10 communities, p_in=(0.2, 0.4), p_out=(0.05, 0.1), time_horizon=8, num_subjects=16, r_time=0.4, r_subject=0.2, setting=2 with subjects as siblings at each time point, SSoS), assigning the 16 subjects to predefined groups such as case vs control. Run MuDCoD.fit_predict as the baseline with alpha=0.05*np.ones((T, 2)) where T=8, beta=0.05*np.ones(S) where S=16, max_K=10, n_iter=30. Modify the β-regularized eigenvector smoothing so it is applied only among subjects within the same predefined group, then rerun fit_predict on the identical adjacency arrays with the same hyper-parameters. Compute adjusted Rand index of both variants against the true labels z_mus_dynamic_true (16 x 8 x 500 arrays), averaging over subjects and time points, and write per-subject/time-point and mean ARI values to ari_comparison.csv.

## Related tools

- **MuDCoD** (Baseline algorithm run via fit_predict; its β-penalty (pulling each subject's smoothed projection matrix toward the mean projection over all other subjects) is the component replaced by group-specific smoothing) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model used to generate grouped multi-subject dynamic networks with known true labels z_mus_dynamic_true for benchmarking)
- **numpy** (Constructs hyper-parameter arrays (alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S)) and provides the multi-dimensional array input format for MuDCoD)

## Examples

```
from mudcod.community_detection import MuDCoD; model = MuDCoD(alpha=0.05*np.ones((8, 2)), beta=0.05*np.ones(16), max_K=10, n_iter=30); pred = model.fit_predict(networks)
```

## Evaluation signals

- Mean ARI (averaged over 16 subjects x 8 time points) of the group-aware variant exceeds the standard MuDCoD baseline ARI on the same simulated networks
- Both variants run on identical simulated adjacency arrays with identical hyper-parameters (alpha=0.05, beta=0.05, max_K=10, n_iter=30), so ARI differences are attributable only to the smoothing change
- Predicted arrays have the same shape as z_mus_dynamic_true (16 x 8 x 500)
- ari_comparison.csv contains both per-subject/time-point values and mean values for both variants
- Cross-validation of alpha and beta (as in examples/cross_validation.py) can be used to confirm the chosen 0.05 values are reasonable

## Limitations

- No concrete numeric ARI results are reported in the article text (the code prints computed values, e.g. 'mean(ARI) of MuDCoD', but no output values appear), so expected ARI ranges must be established empirically
- Evaluation depends entirely on simulated data from MuSDynamicDCBM (setting=2, SSoS: subjects siblings at each time point); results may not transfer to real scRNA-seq-derived networks
- The group-aware smoothing requires predefined subject groupings; mis-specified groups could degrade performance relative to the global-smoothing baseline

## Evidence

- [results] Standard MuDCoD enforces cross-subject signal sharing through the β penalty: [evidence span withheld — anchor status: not_found]
- [methods] Run MuDCoD iterative algorithm to find smoothed spectral representations: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] compare pred_MuDCoD and z_mus_dynamic_true: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [readme] It allow signal sharing between time-steps and subjects by applying eigenvector smoothing: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] simulate_mus_dynamic_dcbm: "mus_dynamic_dcbm.simulate_mus_dynamic_dcbm(setting=1)"
