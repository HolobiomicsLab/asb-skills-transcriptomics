---
name: cross-time-module-similarity-analysis
description: Use when you have per-subject, per-time-point community label predictions
  from a MuDCoD fit on donor-specific scRNA-seq networks and need a cross-time module-persistence
  score per donor, stratified by a donor-level phenotype such as differentiation efficiency.
when_to_use_negative:
- You have not yet run MuDCoD fit_predict — no community label vectors exist to compare,
  so run the community detection skill first.
- Your networks come from a single subject or a single time point — MuDCoD and this
  NMI contrast are designed for multi-subject, multi-time-point personalized dynamic
  networks.
- You need an exact numeric threshold test against the paper — the paper reports no
  numeric NMI cutoff, so only the qualitative ordering (low-efficiency > high-efficiency
  cross-time NMI) is an acceptance criterion, not a score gate.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MuDCoD
  role: Produces the multi-subject, multi-time-point community label vectors (via
    fit_predict with eigenvector smoothing) whose cross-time NMI is computed by this
    skill
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Loads the multi-dimensional numpy array of predicted community labels and
    provides array slicing to extract per-donor day-30 and day-52 label vectors
- name: Dozer
  role: 'Upstream: filters genes and constructs the robust donor-specific co-expression
    networks that MuDCoD consumes before labels are predicted'
  repo: https://github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:ed7429d7b9c1d17255721429df382083d8b1cf4363b24b03fc863432a0fb61fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-time-module-similarity-analysis

## Summary

Quantifies how much a donor's gene co-expression modules persist across time points by computing normalized mutual information (NMI) between the donor's community label vectors at two time points (e.g., day 30 vs day 52) from MuDCoD fit_predict output, then contrasting NMI across donor differentiation-efficiency groups to reproduce the Fig. 5b pattern.

## When to use

Use when you have per-subject, per-time-point community label predictions from a MuDCoD fit on donor-specific scRNA-seq networks and need a cross-time module-persistence score per donor, stratified by a donor-level phenotype such as differentiation efficiency.

## When NOT to use

- You have not yet run MuDCoD fit_predict — no community label vectors exist to compare, so run the community detection skill first.
- Your networks come from a single subject or a single time point — MuDCoD and this NMI contrast are designed for multi-subject, multi-time-point personalized dynamic networks.
- You need an exact numeric threshold test against the paper — the paper reports no numeric NMI cutoff, so only the qualitative ordering (low-efficiency > high-efficiency cross-time NMI) is an acceptance criterion, not a score gate.

## Inputs

- Multi-dimensional numpy array of MuDCoD community label predictions (dimension 1: subjects/donors, dimension 2: time points), saved from a prior MuDCoD fit_predict run on Jerber-2021 donor networks
- Donor differentiation-efficiency grouping (Epen1-defined high/low groups from the prior task)
- Gene co-expression networks per donor per time point, constructed with Dozer filtering from the dopaminergic neuron differentiation scRNA-seq time course

## Outputs

- One NMI value per donor quantifying day-30 vs day-52 module persistence (all values in [0, 1])
- Group-wise NMI summary contrasting within-group-high, within-group-low, and across-group values reproducing the Fig. 5b ordering

## How to apply

Load the multi-dimensional community label predictions produced by the prior MuDCoD fit_predict run (first dimension subjects, second dimension time points) with numpy.load, and for each donor extract the day-30 and day-52 community label vectors from the subject-wise arrays. Compute NMI between each donor's day-30 and day-52 label vectors to yield one cross-time module-persistence value per donor, validating that every NMI lies in [0, 1]. Stratify donors into high versus low differentiation-efficiency groups using the Epen1-defined groups from the prior task, then compare within-group-high and within-group-low NMI values against across-group NMI. The acceptance criterion is a qualitative ordering — low-efficiency donors show higher mean cross-time NMI than high-efficiency donors, and across-group NMI is lower than both within-group values — matching the Fig. 5b contrast; the paper reports no numeric NMI threshold. The label vectors themselves come from MuDCoD's smoothed spectral representations clustered into communities, with alpha/beta chosen by cross-validation as shown in examples/cross_validation.py.

## Related tools

- **MuDCoD** (Produces the multi-subject, multi-time-point community label vectors (via fit_predict with eigenvector smoothing) whose cross-time NMI is computed by this skill) — https://github.com/bo1929/MuDCoD
- **numpy** (Loads the multi-dimensional numpy array of predicted community labels and provides array slicing to extract per-donor day-30 and day-52 label vectors)
- **Dozer** (Upstream: filters genes and constructs the robust donor-specific co-expression networks that MuDCoD consumes before labels are predicted) — https://github.com/shanlu01/Dozer

## Examples

```
import numpy as np; from sklearn.metrics import normalized_mutual_info_score; L = np.load('mudcod_labels.npy'); nmi_donor = normalized_mutual_info_score(L[d]['day30'], L[d]['day52'])
```

## Evaluation signals

- All computed NMI values fall within [0, 1].
- Mean cross-time (day-30 vs day-52) NMI is higher for low-differentiation-efficiency donors than for high-efficiency donors.
- Across-group NMI values are lower than both within-group-high and within-group-low NMI values, matching the reported Fig. 5b contrast.
- Every donor yields exactly one NMI value for each of its day-30/day-52 label-vector pair, with no missing time points in the loaded label array.
- The qualitative ordering is reproducible when alpha/beta hyper-parameters are re-selected via the examples/cross_validation.py procedure.

## Limitations

- The paper reports no numeric NMI threshold — acceptance is only the qualitative ordering of NMI across efficiency groups, so no score gate can be asserted.
- Results depend on MuDCoD hyper-parameters alpha and beta, which should be selected via cross-validation; the article prints computed ARI values but the text contains no concrete numeric results, limiting direct benchmarking.
- Skill presumes labels from the Jerber-2021 dopaminergic neuron differentiation setting with the Epen1-defined high/low efficiency groups; other datasets need their own donor stratification.

## Evidence

- [methods] Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities.: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] MuDCoD takes multi-dimensional numpy arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [readme] MuDCoD provides robust community detection in time-varying personalized networks modules.: "MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules."
- [methods] We suggest to use Dozer to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [methods] You can use cross-validation (see examples/cross_validation.py) for alpha and beta.: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
