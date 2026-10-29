---
name: dynamic-network-community-detection
description: Use when you have personalized adjacency matrices for multiple subjects
  over multiple time points (e.g., S x T x n x n numpy array of gene co-expression
  or similarity networks) and need time-varying, subject-specific community assignments.
when_to_use_negative:
- Networks come from a single subject or a single time point — no multi-subject/multi-time
  smoothing is needed and static spectral clustering suffices
- Input is raw scRNA-seq counts rather than per-subject, per-time-point adjacency
  matrices — network construction (Dozer) must happen first
- Subjects share no apparent information and you want fully independent per-network
  clustering with no cross-subject regularization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: MuDCoD
  role: 'Core algorithm: iterative eigenvector-smoothed spectral community detection
    across subjects and time steps; installed via pip install mudcod or poetry in
    the cloned repository'
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model to generate multi-subject dynamic benchmark networks (e.g.,
    simulate_mus_dynamic_dcbm, setting=2 SSoS) with known true labels for ARI evaluation
  repo: https://github.com/bo1929/MuDCoD
- name: Dozer
  role: R package to filter genes and construct personalized weighted gene co-expression
    networks from population-scale scRNA-seq before applying MuDCoD
  repo: https://github.com/shanlu01/Dozer
- name: numpy
  role: Array format for the S x T x n x n input networks and for building alpha/beta
    hyper-parameter grids
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:3dea1e2a7781406148054e5887e040a5f1b4789e5e98aa54bd29476efd0328aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - PisCES
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# dynamic-network-community-detection

## Summary

Detect communities in personalized dynamic networks (S subjects x T time points) using MuDCoD, which applies eigenvector smoothing across time (alpha) and across subjects (beta) so each subject's spectral representation borrows strength from other subjects at each time point.

## When to use

Use when you have personalized adjacency matrices for multiple subjects over multiple time points (e.g., S x T x n x n numpy array of gene co-expression or similarity networks) and need time-varying, subject-specific community assignments.

## When NOT to use

- Networks come from a single subject or a single time point — no multi-subject/multi-time smoothing is needed and static spectral clustering suffices
- Input is raw scRNA-seq counts rather than per-subject, per-time-point adjacency matrices — network construction (Dozer) must happen first
- Subjects share no apparent information and you want fully independent per-network clustering with no cross-subject regularization

## Inputs

- S x T x n x n multi-dimensional numpy array of adjacency matrices (subjects x time points x nodes x nodes)
- scRNA-seq counts and cell metadata (e.g., Jerber-2021 iPSC dopaminergic differentiation dataset, day-30 and day-52, DA/Sert/Epen1 cell types) for network construction via Dozer
- True multi-subject dynamic community labels (S x T x n array, e.g., z_mus_dynamic_true) when evaluating on simulated data

## Outputs

- Predicted community labels per subject per time point (pred_MuDCoD, S x T arrays of node assignments)
- Smoothed spectral representations of nodes
- Validation metrics: adjusted Rand index (ARI) vs true labels, or pairwise donor-donor NMI per time point with Wilcoxon rank-sum P-values

## How to apply

Assemble the S x T x n x n network array with subjects on the first axis and time points on the second; MuDCoD consumes multi-dimensional numpy arrays directly. For scRNA-seq-derived networks, first build personalized co-expression networks with Dozer (e.g., top 3000 highly variable genes via Scanpy with SCTransform normalization), threshold each weighted network at its top-5% edges, and stop before Dozer's 'Section 4: Gene Centrality Analysis'. Set hyper-parameters alpha (time smoothing, shape (T, 2)) and beta (subject smoothing, one value per subject), e.g., 0.05 * np.ones((T, 2)) and 0.05 * np.ones(S), chosen via cross-validation (examples/cross_validation.py), plus max_K and n_iter (e.g., 30) with monitor_convergence=True. The beta penalty pulls each subject's smoothed projection matrix toward the mean projection matrix over all other subjects at each time point — this term is what enforces cross-subject information sharing and could be made group-specific (case vs control) if subjects form known groups. fit_predict returns smoothed spectral representations clustered into communities, which are validated by comparing pred_MuDCoD against true labels (adjusted Rand index, averaging over subjects and time points) on simulated MuSDynamicDCBM networks, or by pairwise donor-donor NMI on real data.

## Related tools

- **MuDCoD** (Core algorithm: iterative eigenvector-smoothed spectral community detection across subjects and time steps; installed via pip install mudcod or poetry in the cloned repository) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model to generate multi-subject dynamic benchmark networks (e.g., simulate_mus_dynamic_dcbm, setting=2 SSoS) with known true labels for ARI evaluation) — https://github.com/bo1929/MuDCoD
- **Dozer** (R package to filter genes and construct personalized weighted gene co-expression networks from population-scale scRNA-seq before applying MuDCoD) — https://github.com/shanlu01/Dozer
- **numpy** (Array format for the S x T x n x n input networks and for building alpha/beta hyper-parameter grids)

## Examples

```
from mudcod.community_detection import MuDCoD; pred = MuDCoD().fit_predict(adj, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=50, n_iter=30, monitor_convergence=True)
```

## Evaluation signals

- Adjusted Rand index between pred_MuDCoD and z_mus_dynamic_true, averaged over all subjects and time points, on simulated MuSDynamicDCBM networks (e.g., n=500, k=10, p_in=(0.2, 0.4), p_out=(0.05, 0.1), T=8, S=16, r_time=0.4, r_subject=0.2)
- Pairwise donor-donor NMI within each time point should exceed PisCES and per-donor static spectral clustering baselines; on Jerber-2021 day-52 the Wilcoxon rank-sum P-value vs PisCES is reported as 0.0003
- Convergence of the iterative algorithm monitored with monitor_convergence=True over n_iter iterations
- Cross-validation of alpha and beta (examples/cross_validation.py) confirms the chosen regularization strengths (e.g., 0.05) are not arbitrary
- Beta penalty can be ablated: replacing the all-subject mean projection matrix with group-specific means should preserve (or improve) within-group ARI when subjects form known case/control groups

## Limitations

- No concrete numeric ARI values are reported in the article text (the code prints computed values, e.g., mean(ARI) of MuDCoD, but output values do not appear in the document)
- Hyper-parameter selection (alpha, beta) requires cross-validation; defaulting without it risks under- or over-smoothing across subjects and time
- Cross-subject smoothing via beta enforces sharing across ALL subjects toward a global mean, which is inappropriate when subjects form distinct groups (e.g., healthy vs diseased) — this term would need group-specific means

## Evidence

- [readme] eigenvector smoothing rationale (readme): "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] input format (methods): "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] core algorithm (methods): "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] evaluation (methods): "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [methods] upstream network construction (methods): "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [abstract] robustness claim (abstract): "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points"
