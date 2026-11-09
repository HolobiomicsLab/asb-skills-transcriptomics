---
name: differentiation-efficiency-group-contrast-analysis
description: Use when multi-dimensional (subject x time point) community label predictions
  exist from a MuDCoD fit_predict run on donor networks and the goal is to relate
  donor-level cross-time module similarity to a donor phenotype such as differentiation
  efficiency.
when_to_use_negative:
- Input is a single network without both subject and time-point dimensions — MuDCoD's
  setting requires multi-subject, multi-timepoint networks; a static community detection
  method is more appropriate
- No donor-level phenotype grouping (e.g. differentiation efficiency labels) is available,
  so NMI values cannot be stratified into high/low contrast groups
- Goal is gene filtering or co-expression network construction from raw scRNA-seq
  counts rather than contrasting already-computed community labels — use the Dozer
  filtering step instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuDCoD
  role: Produces the multi-subject, multi-timepoint community label predictions (via
    fit_predict with eigenvector smoothing) whose day-30 vs day-52 NMI is computed
    in this skill
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Loads and indexes the multi-dimensional community label arrays; MuDCoD takes
    multi-dimensional numpy arrays as the input network argument
- name: Dozer
  role: Recommended upstream step to filter genes and construct the robust donor co-expression
    networks on which MuDCoD community detection runs
  repo: https://github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:dfdd400fcb724abf13dd4e69947c8ac1ed69e915a2bbbc6b670dc9db02d63384
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differentiation-efficiency-group-contrast-analysis

## Summary

Quantifies cross-time module persistence per iPSC donor by computing normalized mutual information (NMI) between day-30 and day-52 MuDCoD community label vectors, then contrasts NMI values across Epen1-defined high- vs low-differentiation-efficiency donor groups to reproduce the Fig. 5b pattern.

## When to use

Use when multi-dimensional (subject x time point) community label predictions exist from a MuDCoD fit_predict run on donor networks and the goal is to relate donor-level cross-time module similarity to a donor phenotype such as differentiation efficiency.

## When NOT to use

- Input is a single network without both subject and time-point dimensions — MuDCoD's setting requires multi-subject, multi-timepoint networks; a static community detection method is more appropriate
- No donor-level phenotype grouping (e.g. differentiation efficiency labels) is available, so NMI values cannot be stratified into high/low contrast groups
- Goal is gene filtering or co-expression network construction from raw scRNA-seq counts rather than contrasting already-computed community labels — use the Dozer filtering step instead

## Inputs

- Multi-dimensional numpy array of MuDCoD community label predictions (first dimension: subjects/donors, second dimension: time points, e.g. day-30 and day-52)
- Donor-level differentiation-efficiency grouping (Epen1-defined high/low groups from the prior task)
- Donor identifiers mapping rows of the label array to iPSC donors from the Jerber-2021 dopaminergic neuron differentiation scRNA-seq dataset

## Outputs

- One normalized mutual information (NMI) value per donor quantifying day-30 vs day-52 module persistence
- Group-stratified NMI contrast (within-group-high, within-group-low, across-group comparisons) reproducing the Fig. 5b pattern
- Validation report confirming NMI values in [0, 1] and the expected ordering of group means

## How to apply

Load the multi-dimensional community label arrays produced by the prior MuDCoD fit_predict run with numpy.load; MuDCoD natively operates on multi-dimensional numpy arrays whose first dimension is subjects and second dimension is time points. For each donor, extract the day-30 and day-52 community label vectors and compute normalized mutual information between them, writing one NMI value per donor — this NMI is the cross-time module persistence score. Stratify donors by differentiation efficiency using the Epen1-defined high/low groups from the prior task, then compare within-group-high, within-group-low, and across-group NMI values. The paper reports no numeric NMI threshold, so acceptance is the qualitative ordering: low-efficiency donors show higher mean cross-time NMI than high-efficiency donors, with across-group NMI lower than both within-group values, matching the Fig. 5b contrast. All NMI values must lie in [0, 1]; use a library NMI implementation (e.g. sklearn's normalized_mutual_info_score) on integer community labels.

## Related tools

- **MuDCoD** (Produces the multi-subject, multi-timepoint community label predictions (via fit_predict with eigenvector smoothing) whose day-30 vs day-52 NMI is computed in this skill) — https://github.com/bo1929/MuDCoD
- **numpy** (Loads and indexes the multi-dimensional community label arrays; MuDCoD takes multi-dimensional numpy arrays as the input network argument)
- **Dozer** (Recommended upstream step to filter genes and construct the robust donor co-expression networks on which MuDCoD community detection runs) — https://github.com/shanlu01/Dozer

## Examples

```
import numpy as np; from sklearn.metrics import normalized_mutual_info_score; labels = np.load('mudcod_labels.npy'); nmi = normalized_mutual_info_score(labels[d, t_day30], labels[d, t_day52])
```

## Evaluation signals

- All computed NMI values lie in the valid range [0, 1]
- Mean cross-time NMI of low-differentiation-efficiency donors is higher than that of high-efficiency donors
- Across-group NMI is lower than both within-group-high and within-group-low mean NMI values (Fig. 5b qualitative ordering)
- One NMI value is produced per donor, with donor array rows consistent with the subject dimension of the MuDCoD label tensor
- Label vectors at day-30 and day-52 come from the same MuDCoD fit_predict run (same alpha/beta hyperparameters chosen via cross-validation)

## Limitations

- The paper reports no numeric NMI threshold; acceptance relies on the qualitative ordering of group means, not a cutoff
- MuDCoD hyperparameters alpha and beta should be selected via cross-validation (see examples/cross_validation.py), and the choice affects label stability and thus NMI
- MuDCoD performs robustly when there is little or no information sharing among subject networks, but signal sharing via eigenvector smoothing can bias cross-time similarity if donors genuinely differ in module dynamics

## Evidence

- [methods] MuDCoD takes multi-dimensional numpy arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities.: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [abstract] Applications to population-scale scRNA-seq datasets indicate that MuDCoD enables robust inference for identifying time-varying personalized gene modules: "indicate that MuDCoD enables robust inference for identifying time-varying personalized gene modules"
- [readme] It allow signal sharing between time-steps and subjects by applying eigenvector smoothing.: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
