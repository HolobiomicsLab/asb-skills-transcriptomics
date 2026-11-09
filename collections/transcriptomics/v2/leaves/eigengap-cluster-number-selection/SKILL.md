---
name: eigengap-cluster-number-selection
description: Use when running MuDCoD community detection on multi-subject dynamic
  networks (S subjects x T time points adjacency tensors) and the number of communities
  per subject/time must be chosen without a known ground-truth label count.
when_to_use_negative:
- When the number of communities is already known/fixed from prior knowledge — pass
  it rather than estimating via eigengap.
- When input is raw scRNA-seq expression rather than constructed networks — first
  use Dozer to filter genes and build co-expression networks.
- When analyzing a single static network (no subject/time dimensions) — a single-subject
  dynamic spectral method is the wrong setting since MuDCoD requires networks from
  multiple subjects and multiple time points simultaneously.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: MuDCoD
  role: Implements the iterative eigenvector-smoothing spectral algorithm whose eigengap
    heuristic selects K; installed via pip install mudcod
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model (simulate_mus_dynamic_dcbm, setting=1) that generates multi-subject
    dynamic networks with known true labels (z_mus_dynamic_true) for validating eigengap-selected
    K
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Provides the multi-dimensional array format MuDCoD takes as the input network
    argument
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:344a01d063399fec7f93d02b954a668991a42ef5c38e029111af38ce8f5ad745
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# eigengap-cluster-number-selection

## Summary

Selects the number of communities K in MuDCoD's smoothed spectral formulation by locating the eigengap in the spectrum of the degree-normalized Laplacian, optionally bounding the search with a user-supplied max_K.

## When to use

Use when running MuDCoD community detection on multi-subject dynamic networks (S subjects x T time points adjacency tensors) and the number of communities per subject/time must be chosen without a known ground-truth label count.

## When NOT to use

- When the number of communities is already known/fixed from prior knowledge — pass it rather than estimating via eigengap.
- When input is raw scRNA-seq expression rather than constructed networks — first use Dozer to filter genes and build co-expression networks.
- When analyzing a single static network (no subject/time dimensions) — a single-subject dynamic spectral method is the wrong setting since MuDCoD requires networks from multiple subjects and multiple time points simultaneously.

## Inputs

- Multi-dimensional numpy adjacency tensor with first dimension indexing subjects and second indexing time points (e.g., S=16, T=8, n=500 from MuSDynamicDCBM setting=1)
- Hyper-parameters alpha (T x 2 array), beta (S-length array), max_K, n_iter passed to MuDCoD.fit_predict

## Outputs

- Predicted community assignments per subject/time/node (e.g., 16 x 8 x 500 array saved as pred_MuDCoD.npy)
- Implied number of communities K per subject/time point selected by the eigengap of the degree-normalized Laplacian

## How to apply

During MuDCoD's iterative fit_predict, compute the eigendecomposition of the degree-normalized Laplacian of the smoothed spectral representation at each subject/time point and select K at the largest eigengap, up to the max_K bound (e.g., max_K=10 with n=500 simulated nodes). If the true K is being evaluated in simulation, opt_K can be set (e.g., 'null' to disable override) and the eigengap choice is compared against z_mus_dynamic_true accuracy. Because eigengap-based K selection depends on the smoothing strength, tune the alpha (temporal smoothness across adjacent time points) and beta (subject-wise deviation from the mean projection matrix) penalties first — e.g., via the re-sampling network cross-validation of Li et al. 2020 in examples/cross_validation.py, grid-searching alpha=0.05*np.ones((T,2)) and beta=0.05*np.ones(S) grids to maximize the DCBM likelihood. Run with n_iter=30 and monitor_convergence=True so the iterative smoothing-then-eigengap loop is tracked to termination.

## Related tools

- **MuDCoD** (Implements the iterative eigenvector-smoothing spectral algorithm whose eigengap heuristic selects K; installed via pip install mudcod) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model (simulate_mus_dynamic_dcbm, setting=1) that generates multi-subject dynamic networks with known true labels (z_mus_dynamic_true) for validating eigengap-selected K) — https://github.com/bo1929/MuDCoD
- **numpy** (Provides the multi-dimensional array format MuDCoD takes as the input network argument)

## Examples

```
model = MuDCoD(verbose=False); pred = model.fit_predict(adj_tensor, alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K="null", monitor_convergence=True)
```

## Evaluation signals

- Output assignment array exists with the expected S x T x n shape (16 x 8 x 500) and was saved as pred_MuDCoD.npy
- monitor_convergence=True run terminated without error within n_iter=30 iterations
- Compare pred_MuDCoD against z_mus_dynamic_true (e.g., ARI) to confirm eigengap-selected K recovers the simulated k=10 communities under r_time=0.4, r_subject=0.2 signal sharing
- Cross-validation grid over alpha/beta maximizes DCBM likelihood using the Li et al. 2020 re-sampling network cross-validation procedure

## Limitations

- No concrete numeric ARI values are reported in the article text (code prints computed values but outputs are not shown), so expected accuracy ranges must be established empirically.
- Eigengap-based K selection is bounded by max_K (e.g., 10) and may misestimate K when information sharing among subject networks is little or none.
- Requires Python 3.9 environment (conda/pyenv) with poetry-installed dependencies.

## Evidence

- [methods] eigengap K selection in MuDCoD fit_predict: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] alpha/beta penalty semantics: [evidence span withheld — anchor status: not_found]
- [methods] cross-validation for alpha and beta: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
- [readme] eigenvector smoothing purpose: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [abstract] robustness when little sharing: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
- [methods] evaluation against true labels: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
