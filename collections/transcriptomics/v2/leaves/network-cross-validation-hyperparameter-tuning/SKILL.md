---
name: network-cross-validation-hyperparameter-tuning
description: Use when fitting MuDCoD (or similar smoothed spectral community detection)
  to multi-subject dynamic networks and the eigenvector-smoothing penalty strengths
  alpha (over time points) and beta (across subjects) must be tuned rather than guessed.
when_to_use_negative:
- The number of communities K is not determined by this procedure — K is selected
  via the eigengap of the degree-normalized Laplacian within MuDCoD (opt_K/max_K parameters),
  so do not use cross-validation to pick K.
- 'Input is a single-subject or single-time-point network: alpha (temporal smoothness)
  and beta (subject smoothness) penalties are undefined without multiple time points
  and multiple subjects.'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0089
tools:
- name: MuDCoD
  role: Community-detection algorithm whose alpha/beta smoothing hyper-parameters
    are tuned by the cross-validation procedure; cross-validation example lives in
    its examples/cross_validation.py
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model used to generate multi-subject dynamic networks for validating
    hyper-parameter selection
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Formats the multi-dimensional adjacency array (subjects x time points x nodes)
    taken by MuDCoD and defines the per-time-point/per-subject alpha and beta parameter
    arrays
- name: pip
  role: Installs the mudcod package containing the cross-validation utilities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:d0baaa1b652171ef5b4d3a4e0b694c41dcf273c80e2579c19797605444a1e565
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# network-cross-validation-hyperparameter-tuning

## Summary

A re-sampling network cross-validation procedure (Li et al. 2020, as used in MuDCoD's examples/cross_validation.py) that grid-searches the smoothing hyper-parameters alpha and beta by selecting the pair that maximizes the degree-corrected block model (DCBM) likelihood on held-out network data.

## When to use

Use when fitting MuDCoD (or similar smoothed spectral community detection) to multi-subject dynamic networks and the eigenvector-smoothing penalty strengths alpha (over time points) and beta (across subjects) must be tuned rather than guessed.

## When NOT to use

- The number of communities K is not determined by this procedure — K is selected via the eigengap of the degree-normalized Laplacian within MuDCoD (opt_K/max_K parameters), so do not use cross-validation to pick K.
- Input is a single-subject or single-time-point network: alpha (temporal smoothness) and beta (subject smoothness) penalties are undefined without multiple time points and multiple subjects.

## Inputs

- Multi-subject dynamic adjacency tensor: multi-dimensional numpy array with the first dimension indexing subjects and the second indexing time points (e.g. S=16, T=8, n=500 from MuSDynamicDCBM simulation)
- Grid of candidate alpha values (per time point, shape (T, 2)) and beta values (per subject, shape (S,))

## Outputs

- Selected (alpha, beta) pair that maximizes the DCBM likelihood under re-sampling network cross-validation
- Tuned hyper-parameter arrays ready to pass to MuDCoD.fit_predict

## How to apply

Treat alpha and beta as hyper-parameters governing MuDCoD's penalty terms: alpha enforces smoothness of the projection matrices over adjacent time points, while beta constrains subject-specific variation from the mean projection matrix. Assemble the input as a multi-dimensional numpy array whose first dimension indexes subjects and second indexes time points. Grid-search candidate (alpha, beta) pairs using the re-sampling network cross-validation procedure of Li et al. 2020, implemented in examples/cross_validation.py of the MuDCoD repository, and select the pair that maximizes the DCBM likelihood. The selected values are then passed to MuDCoD.fit_predict (e.g. alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S)) together with max_K=10, n_iter=30, and monitor_convergence=True; the number of communities K is separately chosen via the eigengap of the degree-normalized Laplacian rather than by cross-validation.

## Related tools

- **MuDCoD** (Community-detection algorithm whose alpha/beta smoothing hyper-parameters are tuned by the cross-validation procedure; cross-validation example lives in its examples/cross_validation.py) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model used to generate multi-subject dynamic networks for validating hyper-parameter selection) — https://github.com/bo1929/MuDCoD
- **numpy** (Formats the multi-dimensional adjacency array (subjects x time points x nodes) taken by MuDCoD and defines the per-time-point/per-subject alpha and beta parameter arrays)
- **pip** (Installs the mudcod package containing the cross-validation utilities)

## Examples

```
from mudcod.community_detection import MuDCoD; model = MuDCoD(verbose=False).fit_predict(adj_tensor, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K="null", monitor_convergence=True)
```

## Evaluation signals

- The chosen (alpha, beta) pair is the one maximizing DCBM likelihood across the cross-validation grid search.
- Tuned alpha has shape (T, 2) and beta has shape (S); e.g. alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S) for S=16, T=8.
- Downstream MuDCoD.fit_predict with the tuned parameters terminates with monitor_convergence=True without error.
- Predicted assignments have shape S x T x n (16 x 8 x 500) and accuracy can be compared against z_mus_dynamic_true.

## Limitations

- No concrete numeric ARI/likelihood values are reported in the article text, so expected tuning outcomes must be judged by likelihood comparison rather than published reference values.
- The cross-validation procedure follows Li et al. 2020 specifically; other CV schemes are not validated by this article.
- Tuning only covers alpha and beta; community count K is selected by eigengap (max_K bounds the search, opt_K controls selection) and n_iter caps the iterative smoothing.

## Evidence

- [methods] the penalty term with parameter α enforces smoothness of the projection matrices over adjacent time points, while the term with parameter β constrains subject-specific variation: [evidence span withheld — anchor status: not_found]
- [methods] You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`.: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
- [full_text] Grid-search alpha and beta using the re-sampling network cross-validation procedure of Li et al. 2020 ... selecting the pair that maximizes the DCBM likelihood.: [evidence span withheld — anchor status: not_found]
- [full_text] selecting the number of communities K via the eigengap of the degree-normalized Laplacian: [evidence span withheld — anchor status: not_found]
