---
name: statistical-distribution-analysis
description: Use when a per-cell uncertainty or score distribution (e.g. scEMAIL E-scores
  from entropy, confidence, consistency) must be tested for unimodality vs. bimodality
  to make a binary presence/absence decision about novel cell types.
when_to_use_negative:
- The target data are already labeled with known cell types and no novel-type detection
  is required (plain annotation under the closed setting suffices).
- You only have raw scRNA-seq counts without the pre-computed E-score uncertainty
  values — the expert ensemble (entropy, confidence, consistency) must be computed
  first.
- You need to partition cells into known/unknown groups after novelty is confirmed
  — that requires the subsequent adaptive manifold-mixup threshold step, not just
  the bimodality tests.
tools:
- name: scEMAIL
  role: Produces the expert ensemble E-score uncertainty measurements whose distribution
    is tested, and consumes the bimodality decision in its novel cell-type perception
    module
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:1153f1fd98cc408f8e3fbf945d3651ca6717ed505e9761d2248784c750644ff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# statistical-distribution-analysis

## Summary

Applies distribution-shape statistics — the bimodality coefficient and Hartigan's dip test — to an empirical uncertainty distribution (the scEMAIL expert ensemble E-score) to decide whether novel cell types are present in target scRNA-seq data.

## When to use

Use when a per-cell uncertainty or score distribution (e.g. scEMAIL E-scores from entropy, confidence, consistency) must be tested for unimodality vs. bimodality to make a binary presence/absence decision about novel cell types.

## When NOT to use

- The target data are already labeled with known cell types and no novel-type detection is required (plain annotation under the closed setting suffices).
- You only have raw scRNA-seq counts without the pre-computed E-score uncertainty values — the expert ensemble (entropy, confidence, consistency) must be computed first.
- You need to partition cells into known/unknown groups after novelty is confirmed — that requires the subsequent adaptive manifold-mixup threshold step, not just the bimodality tests.

## Inputs

- e_scores.csv: per-cell E-score values computed from the scEMAIL expert ensemble (entropy, confidence, consistency)

## Outputs

- bimodality_decision: JSON file containing the binary decision flag (novel cell type present = 1 / absent = 0) plus the bimodality coefficient and Hartigan's dip test statistic

## How to apply

Load the pre-computed per-cell E-score values from e_scores.csv, computed by the scEMAIL expert ensemble system (entropy EN, confidence CF, and consistency CS). Compute the bimodality coefficient for the empirical E-score distribution using the standard formulation referenced in File S1, and run Hartigan's dip test on the same values. Apply the paper's decision rule: as long as either the bimodality coefficient or the dip test indicates a bimodal distribution, declare the existence of novel cell types (flag = 1); otherwise declare none (flag = 0). The rationale is that when target data contain novel cell types, the empirical distribution of E-scores exhibits an apparent bimodal structure, so distribution shape alone — without a manually defined threshold — signals novelty. Write the binary decision flag plus both test statistics to bimodality_decision. for downstream use by the manifold-mixup adaptive threshold step.

## Related tools

- **scEMAIL** (Produces the expert ensemble E-score uncertainty measurements whose distribution is tested, and consumes the bimodality decision in its novel cell-type perception module) — https://github.com/aster-ww/scEMAIL

## Examples

```
import pandas as pd; e = pd.read_csv('e_scores.csv'); bc = compute_bimodality_coefficient(e['E_score']); dip_stat, p = hartigan_dip_test(e['E_score']); flag = 1 if (bc_flag or dip_flag) else 0
```

## Evaluation signals

- bimodality_decision. exists, parses as valid JSON, and contains both the binary decision field and the two test statistics (bimodality coefficient, Hartigan's dip test).
- The binary flag is consistent with the either-test-positive rule: flag = 1 if and only if at least one of the two tests indicates a bimodal distribution.
- As a sanity check, E-score distributions on target datasets known to contain novel cell types (e.g. the open-setting datasets such as mammary gland and neonatal rib) should show apparent bimodal structure and yield flag = 1.
- The decision flag correctly triggers the downstream adaptive manifold-mixup threshold step only when novelty was declared.

## Limitations

- The decision is binary and based on aggregate distribution shape: a bimodal test outcome declares only the existence of novel cell types and does not identify how many, or which, cells are novel — partitioning is handled separately by the adaptive threshold.
- Implementation details of the bimodality tests (the exact formulation in File S1) are deferred to the supplementary material and are not given in the main text.
- The empirical bimodal structure of E-scores is reported as an empirical observation for target data with novel cell types; mixed or partially-overlapping cell populations may weaken the separation.

## Evidence

- [readme] Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types.: "Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types."
- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
