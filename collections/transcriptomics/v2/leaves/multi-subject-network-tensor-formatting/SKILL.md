---
name: multi-subject-network-tensor-formatting
description: Use when preparing adjacency matrices from multiple subjects sampled
  across multiple time points for MuDCoD fit_predict — input must be a numpy tensor
  with subject as the first dimension and time point as the second.
when_to_use_negative:
- Input is a single network or single-subject time series — MuDCoD's tensor format
  assumes networks from multiple subjects and multiple time points simultaneously,
  which no pre-existing community detection setting included
- Raw scRNA-seq count data that has not yet been filtered and turned into gene co-expression
  networks by Dozer — tensor assembly presumes per-subject adjacency matrices already
  exist
- Data stored in non-numpy structures (e.g., a list of R matrices) — must be converted
  to numpy arrays before MuDCoD, which takes multi-dimensional numpy arrays as the
  input network argument
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: MuDCoD
  role: Consumes the S x T x n numpy adjacency tensor as its input network argument
    for multi-subject dynamic community detection via eigenvector smoothing
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model that generates the multi-subject dynamic DCBM adjacency series
    to be assembled into the tensor
  repo: https://github.com/bo1929/MuDCoD
- name: Dozer
  role: R package used to filter genes and construct robust personalized gene co-expression
    networks (output as .rda) that then feed the tensor for real scRNA-seq data
  repo: https://github.com/shanlu01/Dozer
- name: numpy
  role: Provides the multi-dimensional array structure MuDCoD requires for its input
    network argument
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:09eeb89adfb31b67ef1ec90911d057f435d2db9d306960b1a444233b6204923d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# multi-subject-network-tensor-formatting

## Summary

Formats multi-subject dynamic networks (e.g., simulated via MuSDynamicDCBM or built from scRNA-seq gene co-expression networks) into the multi-dimensional numpy adjacency tensor (S x T x n) that MuDCoD requires as input for community detection.

## When to use

Use when preparing adjacency matrices from multiple subjects sampled across multiple time points for MuDCoD fit_predict — input must be a numpy tensor with subject as the first dimension and time point as the second.

## When NOT to use

- Input is a single network or single-subject time series — MuDCoD's tensor format assumes networks from multiple subjects and multiple time points simultaneously, which no pre-existing community detection setting included
- Raw scRNA-seq count data that has not yet been filtered and turned into gene co-expression networks by Dozer — tensor assembly presumes per-subject adjacency matrices already exist
- Data stored in non-numpy structures (e.g., a list of R matrices) — must be converted to numpy arrays before MuDCoD, which takes multi-dimensional numpy arrays as the input network argument

## Inputs

- Per-subject, per-time-point adjacency matrices (S x T collection of n x n networks) from simulate_mus_dynamic_dcbm or Dozer-constructed gene co-expression networks
- Networks exported from Dozer as an .rda file (real scRNA-seq data path)

## Outputs

- Multi-dimensional numpy array (tensor) with first dimension indexing subjects, second indexing time points, and node dimension of size n (e.g., 16 x 8 x 500), suitable as the MuDCoD fit_predict network argument

## How to apply

First obtain the per-subject, per-time-point adjacency matrices — either by simulating a multi-subject dynamic degree-corrected block model series with MuSDynamicDCBM via simulate_mus_dynamic_dcbm(setting=1), or by using Dozer to filter genes and construct personalized gene co-expression networks from scRNA-seq data (following the Dozer vignette through 'Section 4: Gene Centrality Analysis' to export networks as an .rda file). Then assemble the adjacency arrays into a multi-dimensional numpy array where the first dimension indexes subjects and the second indexes time points, with the remaining dimension(s) holding the n nodes — e.g., for S=16 subjects, T=8 time points, n=500 nodes the tensor is 16 x 8 x 500. MuDCoD takes these multi-dimensional numpy arrays directly as its input network argument, so no additional reshaping is needed once dimensions are ordered subject → time → node. Verify dimensionality before passing to MuDCoD(verbose=False).fit_predict, since the predicted assignments are returned in the same S x T x n layout (e.g., 16 x 8 x 500 saved to pred_MuDCoD.npy).

## Related tools

- **MuDCoD** (Consumes the S x T x n numpy adjacency tensor as its input network argument for multi-subject dynamic community detection via eigenvector smoothing) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model that generates the multi-subject dynamic DCBM adjacency series to be assembled into the tensor) — https://github.com/bo1929/MuDCoD
- **Dozer** (R package used to filter genes and construct robust personalized gene co-expression networks (output as .rda) that then feed the tensor for real scRNA-seq data) — https://github.com/shanlu01/Dozer
- **numpy** (Provides the multi-dimensional array structure MuDCoD requires for its input network argument)

## Examples

```
from mus_dynamic_dcbm import simulate_mus_dynamic_dcbm; adj = simulate_mus_dynamic_dcbm(setting=1); A = np.asarray(adj)  # shape (S, T, n)
```

## Evaluation signals

- Assembled tensor has shape (S, T, n) matching the source data, e.g., 16 x 8 x 500 for num_subjects=16, time_horizon=8, n=500
- Output of fit_predict preserves the same S x T x n layout: pred_MuDCoD.npy exists with shape 16 x 8 x 500
- Predicted assignments can be directly compared elementwise with z_mus_dynamic_true to evaluate accuracy
- MuDCoD fit_predict runs with monitor_convergence=True and terminates without error on the assembled tensor

## Limitations

- MuDCoD takes multi-dimensional numpy arrays as the input network argument, so data must be in numpy format; R-produced Dozer networks (.rda) require conversion before use
- Requires Python version 3.9 environment (created with conda or pyenv) with dependencies installed via poetry or pip install mudcod
- The paper's text reports no concrete numeric ARI values (the code prints computed values but output values do not appear in text), so accuracy expectations must be established empirically against true labels

## Evidence

- [methods] MuDCoD takes multi-dimensional `numpy` arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [full_text] Assemble adjacency arrays into tensor with subject-first, time-second dimension ordering: [evidence span withheld — anchor status: not_found]
- [methods] Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities.: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [readme] MuDCoD provides robust community detection in time-varying personalized network modules via eigenvector smoothing: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
