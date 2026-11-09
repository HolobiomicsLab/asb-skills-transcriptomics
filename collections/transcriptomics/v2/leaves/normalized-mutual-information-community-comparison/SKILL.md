---
name: normalized-mutual-information-community-comparison
description: Use when you have per-subject, per-time-point community label vectors
  from a multi-subject dynamic community detection run (e.g., MuDCoD fit_predict)
  and need to measure cross-time or cross-condition module similarity per subject
  and compare it against subject-level phenotypes.
when_to_use_negative:
- You have raw expression matrices or networks but no community label assignments
  yet — run MuDCoD community detection (or equivalent) first; NMI here compares predicted
  community labels, not raw data
- You need to compare communities across different sets of nodes/genes — NMI on label
  vectors assumes the same node set in both labelings
- You are evaluating clustering accuracy against known ground-truth labels in a simulation
  — the MuDCoD workflow uses ARI for that comparison (e.g., pred_MuDCoD vs. z_mus_dynamic_true)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3473
tools:
- name: MuDCoD
  role: Source of the per-subject, per-time-point community label vectors that are
    compared via NMI; run via fit_predict on multi-dimensional numpy network inputs
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Loads the multi-dimensional community label prediction arrays (numpy.load)
    and holds the label vectors used in NMI computation
- name: Dozer
  role: 'Upstream: filters genes and constructs the robust personalized gene co-expression
    networks (output to .rda) that MuDCoD community detection is run on'
  repo: https://github.com/shanlu01/Dozer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:f2b80486722dfcf064cea080c494633b9dc99e2cf3b6c334723e27e90ecb31d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - numpy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# normalized-mutual-information-community-comparison

## Summary

Quantifies persistence of gene co-expression community assignments across conditions or time points by computing normalized mutual information (NMI) between community label vectors, then contrasting NMI values across donor/subject groups (e.g., differentiation-efficiency high vs. low) to reproduce Fig. 5b-style module-similarity contrasts.

## When to use

Use when you have per-subject, per-time-point community label vectors from a multi-subject dynamic community detection run (e.g., MuDCoD fit_predict) and need to measure cross-time or cross-condition module similarity per subject and compare it against subject-level phenotypes.

## When NOT to use

- You have raw expression matrices or networks but no community label assignments yet — run MuDCoD community detection (or equivalent) first; NMI here compares predicted community labels, not raw data
- You need to compare communities across different sets of nodes/genes — NMI on label vectors assumes the same node set in both labelings
- You are evaluating clustering accuracy against known ground-truth labels in a simulation — the MuDCoD workflow uses ARI for that comparison (e.g., pred_MuDCoD vs. z_mus_dynamic_true)

## Inputs

- Multi-dimensional numpy array of community label predictions from MuDCoD fit_predict (first dimension: subjects, second dimension: time points)
- Per-donor community label vectors for the two time points being compared (e.g., day-30 and day-52)
- Donor phenotype stratification (e.g., Epen1-defined high/low differentiation-efficiency groups)

## Outputs

- One NMI value per donor quantifying cross-time module persistence (values in [0, 1])
- Group-level NMI contrast (within-group-high, within-group-low, across-group) reproducing the Fig. 5b ordering

## How to apply

Load the multi-dimensional community label predictions (first dimension subjects, second dimension time points) produced by a prior MuDCoD fit_predict run using numpy.load. For each donor, extract the two community label vectors to compare (e.g., day-30 and day-52 from the Jerber-2021 dopaminergic neuron differentiation time course) and compute normalized mutual information between them, writing one NMI value per donor. Because NMI lies in [0, 1], first validate all values fall in that range. Stratify donors by the phenotype of interest (e.g., Epen1-defined high/low differentiation-efficiency groups) and contrast within-group-high and within-group-low NMI values against across-group NMI values. The acceptance criterion is a qualitative ordering — low-efficiency donors show higher mean cross-time NMI than high-efficiency donors, with across-group NMI lower than both within-group values — matching the reported Fig. 5b contrast, since the paper reports no numeric NMI threshold.

## Related tools

- **MuDCoD** (Source of the per-subject, per-time-point community label vectors that are compared via NMI; run via fit_predict on multi-dimensional numpy network inputs) — https://github.com/bo1929/MuDCoD
- **numpy** (Loads the multi-dimensional community label prediction arrays (numpy.load) and holds the label vectors used in NMI computation)
- **Dozer** (Upstream: filters genes and constructs the robust personalized gene co-expression networks (output to .rda) that MuDCoD community detection is run on) — https://github.com/shanlu01/Dozer

## Examples

```
labels = numpy.load('mudcod_predictions.npy'); nmi_30_52 = normalized_mutual_info_score(labels[d, 30], labels[d, 52])
```

## Evaluation signals

- All computed NMI values lie in [0, 1]
- Low-differentiation-efficiency donors exhibit higher mean cross-time (day-30 vs day-52) NMI than high-efficiency donors
- Across-group NMI is lower than both within-group (high and low) values, matching the reported Fig. 5b qualitative ordering
- Each donor yields exactly one NMI value per time-point pair; label vector lengths match the number of genes/nodes in the network

## Limitations

- The paper reports no numeric NMI threshold — acceptance is the qualitative ordering of group means, not a cutoff
- Underlying community assignments depend on MuDCoD hyper-parameters (alpha, beta, max_K, n_iter); alpha/beta should be chosen via cross-validation (examples/cross_validation.py)
- No concrete numeric NMI results are present in the article text; only qualitative patterns are verifiable from the source

## Evidence

- [other] Compute normalized mutual information (NMI) between each donor's day-30 and day-52 label vectors to quantify cross-time module persistence: [evidence span withheld — anchor status: not_found]
- [methods] MuDCoD takes multi-dimensional numpy arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [other] low-efficiency donors show higher cross-time module similarity: [evidence span withheld — anchor status: not_found]
- [methods] Run MuDCoD fit_predict to find smoothed spectral representations: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] hyper-parameters alpha, beta via cross-validation: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
- [readme] README citation of MuDCoD purpose: "MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules."
