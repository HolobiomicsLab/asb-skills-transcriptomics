---
name: hartigan-dip-test-statistics
description: Use when a per-cell uncertainty score distribution (e.g. scEMAIL E-scores
  from entropy, confidence, consistency) must be tested for bimodality to declare
  the presence of novel cell types without user-defined thresholds.
when_to_use_negative:
- The E-score distribution is already known to be unimodal and novel cell types have
  been ruled out or are irrelevant to the analysis goal
- A novel-cell-type presence decision cannot be reduced to a binary flag — use the
  downstream adaptive manifold-mixup threshold step instead once bimodality is confirmed
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Produces the E-score uncertainty measurements and consumes the bimodality
    decision as step two of its novel cell-type perception module
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:0349f80af85f7077c06f9b7a5faa7949939cab4665591bbcedc4e8b6974e48f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hartigan-dip-test-statistics

## Summary

Applies bimodality statistics — the bimodality coefficient and Hartigan's dip test — to the distribution of scEMAIL's expert ensemble (E-score) uncertainty values to decide, via an either-test-positive rule, whether novel cell types are present in target scRNA-seq data.

## When to use

Use when a per-cell uncertainty score distribution (e.g. scEMAIL E-scores from entropy, confidence, consistency) must be tested for bimodality to declare the presence of novel cell types without user-defined thresholds.

## When NOT to use

- The E-score distribution is already known to be unimodal and novel cell types have been ruled out or are irrelevant to the analysis goal
- A novel-cell-type presence decision cannot be reduced to a binary flag — use the downstream adaptive manifold-mixup threshold step instead once bimodality is confirmed

## Inputs

- Per-cell E-score uncertainty values for the target dataset (e_scores.csv), computed from entropy (EN), confidence (CF), and consistency (CS) via the scEMAIL expert ensemble

## Outputs

- bimodality_decision. containing the binary novel-cell-type decision flag plus the bimodality coefficient and Hartigan dip test statistic

## How to apply

Load pre-computed per-cell E-score values (combining entropy EN, confidence CF, and consistency CS from the scEMAIL expert ensemble) for the target dataset from e_scores.csv. Compute the bimodality coefficient using the standard formulation referenced in File S1, and run Hartigan's dip test on the same E-score values to assess unimodality vs. bimodality. Apply the decision rule that if either the bimodality coefficient or the dip test indicates a bimodal distribution, set the novel-cell-type flag to 1; otherwise set it to 0. The rationale, per the paper, is that when target data contain novel cell types the empirical E-score distribution exhibits apparent bimodal structure, so bimodality tests detect new types automatically with no prior information or manual thresholds. Write the binary decision flag plus both test statistics to bimodality_decision..

## Related tools

- **scEMAIL** (Produces the E-score uncertainty measurements and consumes the bimodality decision as step two of its novel cell-type perception module) — https://github.com/aster-ww/scEMAIL

## Examples

```
dip.test(read.csv('e_scores.csv')$e_score)  # combine with bimodality coefficient; flag = 1 if either indicates bimodality
```

## Evaluation signals

- bimodality_decision. exists and is valid JSON
- Output contains both the binary decision field and the two test statistics (bimodality coefficient and dip test)
- The decision is consistent with the either-test-positive rule: flag = 1 if either statistic indicates bimodality, flag = 0 otherwise
- Test statistics are computed on the same per-cell E-score values loaded from e_scores.csv (cell count matches the target dataset)
- E-score distributions for targets with confirmed novel cell types visually show apparent bimodal structure, matching the paper's reported behavior

## Limitations

- The paper defers detailed statistical formulations and implementation details to File S1, so exact p-value cutoffs for the dip test are not stated in the main text
- The decision is binary (novel types present or absent); separating which specific cells are 'unknown' requires the subsequent adaptive manifold-mixup threshold step

## Evidence

- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [readme] Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types.: "Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types."
