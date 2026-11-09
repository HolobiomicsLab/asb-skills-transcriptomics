---
name: subject-smoothing-regularization-modification
description: Use when analyzing multi-subject dynamic networks with MuDCoD where subjects
  fall into natural groups (e.g., healthy vs diseased), and the global β subject-smoothing
  term inappropriately pulls subjects toward an all-subject mean.
when_to_use_negative:
- Subjects show no apparent grouping structure — standard MuDCoD already performs
  robustly when there is no or little information sharing among the networks, and
  a global mean is appropriate
- Single-subject or single-time-point networks — the β subject-smoothing term only
  exists in the multi-subject (S x T) setting and there is no cross-subject mean to
  replace
- Group labels are unknown or unreliable — group-specific smoothing requires predefined
  groups; without them the extension cannot be parameterized
edam_operation: http://edamontology.org/operation_3432
tools:
- name: MuDCoD
  role: Iterative algorithm whose β-regularized eigenvector subject-smoothing term
    is modified to use group-specific means; fit_predict produces community predictions
    for both variants
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model used to generate grouped multi-subject dynamic networks (setting=2,
    SSoS) for benchmarking the group-aware extension against standard MuDCoD
- name: numpy
  role: Provides the multi-dimensional arrays passed as the input network argument
    to MuDCoD and used to construct alpha/beta hyper-parameter arrays
- name: Dozer
  role: Recommended upstream step to filter genes and construct robust scRNA-seq co-expression
    networks before applying MuDCoD-style community detection
  repo: https://github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:f74f97d6a999f11baa4bf8e69d65e6ca8d1a4c42c49e3422a143ee5df179ba05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# subject-smoothing-regularization-modification

## Summary

A MuDCoD extension that replaces the β-penalty's global all-subject mean projection matrix with group-specific means (e.g., case vs control), so eigenvector smoothing enforces cross-subject information sharing only within predefined subject groups rather than across all subjects.

## When to use

Use when analyzing multi-subject dynamic networks with MuDCoD where subjects fall into natural groups (e.g., healthy vs diseased), and the global β subject-smoothing term inappropriately pulls subjects toward an all-subject mean.

## When NOT to use

- Subjects show no apparent grouping structure — standard MuDCoD already performs robustly when there is no or little information sharing among the networks, and a global mean is appropriate
- Single-subject or single-time-point networks — the β subject-smoothing term only exists in the multi-subject (S x T) setting and there is no cross-subject mean to replace
- Group labels are unknown or unreliable — group-specific smoothing requires predefined groups; without them the extension cannot be parameterized

## Inputs

- Multi-subject dynamic network array of shape S x T x n x n (S=16 subjects, T=8 time points, n=500 nodes), e.g. from MuSDynamicDCBM simulation (setting=2, subjects siblings at each time point / SSoS)
- Predefined subject group assignments (e.g., case vs control labels for the 16 subjects)
- True community labels z_mus_dynamic_true (16 x 8 x 500) for evaluation
- Hyper-parameters: alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S), max_K=10, n_iter=30

## Outputs

- Predicted communities for the standard MuDCoD and the group-aware variant (fit_predict output)
- Per-subject/time-point and mean adjusted Rand index values for both variants
- ari_comparison.csv containing ARI values for standard MuDCoD vs the group-aware extension

## How to apply

Identify the β penalty as the MuDCoD objective component responsible for cross-subject sharing: it pulls each subject's smoothed projection matrix toward the mean projection matrix computed over all other subjects at each time point. Replace this global mean with group-specific means so β-regularized eigenvector smoothing is applied only among subjects within the same predefined group (e.g., case vs control). Validate on simulated grouped networks from MuSDynamicDCBM (n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1), time_horizon=8, num_subjects=16, r_time=0.4, r_subject=0.2, setting=2/SSoS), holding hyper-parameters fixed (alpha=0.05*np.ones((T,2)) with T=8, beta=0.05*np.ones(S) with S=16, max_K=10, n_iter=30) across both variants. Compare predicted communities to true labels z_mus_dynamic_true (16 x 8 x 500 arrays) via adjusted Rand index averaged over subjects and time points, and write per-subject/time-point plus mean ARI values for standard and group-aware variants to ari_comparison.csv.

## Related tools

- **MuDCoD** (Iterative algorithm whose β-regularized eigenvector subject-smoothing term is modified to use group-specific means; fit_predict produces community predictions for both variants) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model used to generate grouped multi-subject dynamic networks (setting=2, SSoS) for benchmarking the group-aware extension against standard MuDCoD)
- **numpy** (Provides the multi-dimensional arrays passed as the input network argument to MuDCoD and used to construct alpha/beta hyper-parameter arrays)
- **Dozer** (Recommended upstream step to filter genes and construct robust scRNA-seq co-expression networks before applying MuDCoD-style community detection) — https://github.com/shanlu01/Dozer

## Examples

```
pred = MuDCoD.fit_predict(A_SxTxnxn, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30); ari = adjusted_rand_score(pred, z_mus_dynamic_true)
```

## Evaluation signals

- Adjusted Rand index of the group-aware variant against z_mus_dynamic_true (16 x 8 x 500) should improve over or match standard MuDCoD when averaged over subjects and time points
- Mean ARI values for both variants are written to ari_comparison.csv with per-subject/time-point breakdowns, enabling diff-style comparison of the two variants on identical simulated networks
- Both standard and group-aware runs must use identical hyper-parameters (alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30) and identical simulated adjacency arrays — any ARI difference is attributable to the smoothing modification
- Sanity check that the modified smoothing pulls each subject only toward within-group means, not the all-subject mean, at each time point

## Limitations

- Requires predefined subject groupings; the article's evaluation used simulated groups (case vs control) with known labels, so real-world group assignments must be trusted inputs
- No concrete numeric ARI results are reported in the article text (code prints computed values but no output values appear in text), so expected performance gains are not numerically grounded here
- The modified smoothing replaces the global β penalty term — if group sizes are very small, within-group mean projection matrices may be noisy

## Evidence

- [readme] cross-subject smoothing mechanism: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] MuDCoD core procedure being modified: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] evaluation against ground truth: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [methods] simulation input construction: "First construct a `MuSDynamicDCBM` instance, i.e., simulation model, with desired parameters."
- [intro] sharing rationale: "When available, MuDCoD leverages common signals among networks of the subjects and performs robustly when subjects do not share any apparent information."
