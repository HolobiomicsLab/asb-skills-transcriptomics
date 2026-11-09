---
name: spectral-community-detection
description: Use when the input is a multi-dimensional adjacency tensor (S subjects
  x T time points, e.g. a 16 x 8 x 500 numpy array) and the goal is to infer time-varying
  personalized network modules, especially when subjects share little or no apparent
  information.
when_to_use_negative:
- The input is a single static network or a single-subject time series — use static
  spectral clustering or single-subject dynamic baselines (PisCES, Betzel-2019) instead,
  as MuDCoD targets networks from multiple subjects and multiple time points simultaneously
- 'The input is raw scRNA-seq count data — construct gene co-expression networks first
  with Dozer (follow its vignette up to ''Section 4: Gene Centrality Analysis'' to
  output networks in an .rda file)'
- The input is not a multi-dimensional numpy array network tensor, since MuDCoD takes
  multi-dimensional numpy arrays as the input network argument
edam_operation: http://edamontology.org/operation_3432
tools:
- name: MuDCoD
  role: 'Core algorithm: iterative eigenvector-smoothed spectral community detection
    across subjects and time points via fit_predict'
  repo: github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model generating multi-subject dynamic degree-corrected block model
    ground-truth networks and labels (z_mus_dynamic_true) for benchmarking accuracy
  repo: github.com/bo1929/MuDCoD
- name: numpy
  role: 'Input format: the adjacency tensor is a multi-dimensional numpy array; alpha
    and beta penalty arrays are numpy arrays'
- name: Dozer
  role: Recommended upstream tool to filter genes and construct robust personalized
    gene co-expression networks from scRNA-seq data before community detection
  repo: github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:63e7977b0685a619e4ff11d72656d2f56962456066917a7482923a96a6487bbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-community-detection

## Summary

Detects communities in multi-subject dynamic networks by computing smoothed spectral representations of nodes across S subjects and T time points, then clustering them. MuDCoD's eigenvector smoothing shares signal across adjacent time points (alpha) and toward the subject-mean projection matrix (beta), enabling robust community detection in personalized dynamic gene networks.

## When to use

Use when the input is a multi-dimensional adjacency tensor (S subjects x T time points, e.g. a 16 x 8 x 500 numpy array) and the goal is to infer time-varying personalized network modules, especially when subjects share little or no apparent information.

## When NOT to use

- The input is a single static network or a single-subject time series — use static spectral clustering or single-subject dynamic baselines (PisCES, Betzel-2019) instead, as MuDCoD targets networks from multiple subjects and multiple time points simultaneously
- The input is raw scRNA-seq count data — construct gene co-expression networks first with Dozer (follow its vignette up to 'Section 4: Gene Centrality Analysis' to output networks in an .rda file)
- The input is not a multi-dimensional numpy array network tensor, since MuDCoD takes multi-dimensional numpy arrays as the input network argument

## Inputs

- Multi-dimensional numpy adjacency tensor of shape (S, T, n) — subjects x time points x nodes (networks can be built from scRNA-seq with Dozer)
- MuSDynamicDCBM simulation parameters: n=500, k=10, num_subjects=16, p_in=(0.2,0.4), p_out=(0.05,0.1), r_time, r_subject, time_horizon T in {2,4,8}
- Hyperparameters alpha (shape (T,2)) and beta (shape (S)), optionally selected by network cross-validation

## Outputs

- Predicted community assignments array of shape (S, T, n), e.g. a 16 x 8 x 500 array saved as pred_MuDCoD.npy
- Per-subject, per-time-point adjusted Rand index scores against true labels (aggregated mean ARI, e.g. written to ari_results.csv)

## How to apply

Assemble the input as a multi-dimensional numpy array with the first dimension indexing subjects and the second indexing time points. Optionally tune the smoothness penalties alpha (over adjacent time points) and beta (toward the mean projection matrix across subjects) via the re-sampling network cross-validation procedure of Li et al. 2020 that maximizes the DCBM likelihood. Run MuDCoD's iterative fit_predict (e.g. alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K='null', monitor_convergence=True) to iteratively compute smoothed spectral representations of nodes, selecting the number of communities K via the eigengap of the degree-normalized Laplacian, then cluster these representations into communities. Evaluate by comparing the predicted assignments against true labels (e.g. z_mus_dynamic_true) using the adjusted Rand index per subject and time point. For simulated benchmarks, generate ground truth with MuSDynamicDCBM via simulate_mus_dynamic_dcbm (n=500, k=10, p_in=(0.2,0.4), p_out=(0.05,0.1)) and sweep r_time, r_subject, and T in {2,4,8}.

## Related tools

- **MuDCoD** (Core algorithm: iterative eigenvector-smoothed spectral community detection across subjects and time points via fit_predict) — github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model generating multi-subject dynamic degree-corrected block model ground-truth networks and labels (z_mus_dynamic_true) for benchmarking accuracy) — github.com/bo1929/MuDCoD
- **numpy** (Input format: the adjacency tensor is a multi-dimensional numpy array; alpha and beta penalty arrays are numpy arrays)
- **Dozer** (Recommended upstream tool to filter genes and construct robust personalized gene co-expression networks from scRNA-seq data before community detection) — github.com/shanlu01/Dozer

## Examples

```
from mudcod import MuDCoD; pred = MuDCoD(verbose=False).fit_predict(adj, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K='null', monitor_convergence=True)
```

## Evaluation signals

- Output array exists with shape S x T x n (e.g. 16 x 8 x 500) and convergence monitoring terminated without error
- Adjusted Rand index computed per subject and time point between pred_MuDCoD and z_mus_dynamic_true; mean ARI aggregated across subjects, time points, and replicates
- Reproduced benchmark curves match Fig. 3 patterns: MuDCoD achieves a >20% mean-ARI gain over PisCES at SSoT with r_time=0 and T=8, and Betzel-2019 yields mean ARI <0.2 at r_subject=0.5 and/or r_time=0.5
- Number of communities K selected by the eigengap of the degree-normalized Laplacian stays within max_K=10
- For scRNA-seq applications, networks passed in were produced by the Dozer pipeline rather than raw counts

## Limitations

- No concrete numeric ARI values are reported in the article text (the code prints computed mean ARI values but the output values do not appear), so benchmark accuracy must be verified by reproduction
- Alpha and beta require tuning via the re-sampling network cross-validation procedure of Li et al. 2020 (examples/cross_validation.py); defaults such as alpha=0.05*np.ones((T,2)) and beta=0.05*np.ones(S) are only starting points
- In the benchmark invocation opt_K='null' is used, meaning automatic K selection is disabled and K handling must be managed explicitly

## Evidence

- [readme] README: eigenvector smoothing across subjects and time: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] Methods: fit_predict computes smoothed spectral representations then clusters: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] Methods: input format requirement: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] Methods: accuracy evaluation against ground truth: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [abstract] Abstract: robustness when subjects share little information: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
