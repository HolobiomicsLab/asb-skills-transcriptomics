---
name: bimodality-testing-of-uncertainty-scores
description: Use when annotating target scRNA-seq cells with a pre-trained source
  model and you must decide—without manual thresholds or prior information—whether
  the target contains unseen (novel) cell types based on per-cell uncertainty scores.
when_to_use_negative:
- The target data contain only known cell types already covered by the source label
  set and no novel-type screening is needed
- Per-cell uncertainty scores (E-scores) have not yet been computed from the expert
  ensemble (entropy, confidence, consistency)—run the E-score computation step first
- A user-supplied novelty threshold already exists and manual thresholding is acceptable—scEMAIL's
  point is to avoid subjective thresholds
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Source framework whose expert ensemble produces the E-scores and whose novel
    cell-type perception module applies the bimodality tests
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:c64a72bd00b621b60536ff4c1804901a2790c1603e301de4bb9af1c6a9571790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bimodality-testing-of-uncertainty-scores

## Summary

Decision procedure from scEMAIL that tests the per-cell E-score uncertainty distribution (from entropy, confidence, and consistency in an expert ensemble) for bimodality to automatically judge whether novel cell types are present in target scRNA-seq data.

## When to use

Use when annotating target scRNA-seq cells with a pre-trained source model and you must decide—without manual thresholds or prior information—whether the target contains unseen (novel) cell types based on per-cell uncertainty scores.

## When NOT to use

- The target data contain only known cell types already covered by the source label set and no novel-type screening is needed
- Per-cell uncertainty scores (E-scores) have not yet been computed from the expert ensemble (entropy, confidence, consistency)—run the E-score computation step first
- A user-supplied novelty threshold already exists and manual thresholding is acceptable—scEMAIL's point is to avoid subjective thresholds

## Inputs

- Per-cell E-score uncertainty values for the target scRNA-seq dataset (e.g. e_scores.csv), produced by the scEMAIL expert ensemble (entropy, confidence, consistency)

## Outputs

- Binary decision flag (1 = novel cell type present, 0 = none) with the bimodality coefficient and Hartigan's dip test statistic (e.g. bimodality_decision.)

## How to apply

Load the per-cell E-score values for the target dataset (e.g. from e_scores.csv); these scores combine three complementary indices—entropy (EN), confidence (CF), and consistency (CS)—computed by the scEMAIL expert ensemble system. Compute the bimodality coefficient for the empirical E-score distribution as referenced in File S1, and run Hartigan's dip test on the same values to assess unimodality vs. bimodality. Apply scEMAIL's decision rule: as long as one of these measures indicates a bimodal structure, declare the existence of novel cell types (binary flag = 1); otherwise flag = 0. The rationale is that when target data contain novel cell types, their empirical distribution of E-score(z_it) exhibits an apparent bimodal structure. Record both test statistics plus the binary decision so downstream steps (adaptive manifold-mixup thresholding) can proceed only when novel types are confirmed.

## Related tools

- **scEMAIL** (Source framework whose expert ensemble produces the E-scores and whose novel cell-type perception module applies the bimodality tests) — https://github.com/aster-ww/scEMAIL

## Examples

```
import pandas as pd; from diptest import diptest; e = pd.read_csv('e_scores.csv')['e_score']; stat, p = diptest(e); flag = int(p < 0.05)
```

## Evaluation signals

- bimodality_decision output file exists and is valid JSON containing both the binary decision field and the two test statistics
- The binary decision matches the either-test-positive rule: flag = 1 exactly when the bimodality coefficient or Hartigan's dip test indicates a bimodal distribution
- Simulated/synthetic E-score distributions with a known unimodal shape (no novel types) yield flag = 0
- E-scores from a target dataset known to contain held-out cell types yield an apparently bimodal distribution and flag = 1

## Limitations

- Implementation details of the tests are deferred to supplementary File S1 rather than fully specified in the main text
- The decision is binary ('as long as one of these measures indicates the bimodal structure')—it detects presence of novel types but does not enumerate or label how many novel types exist

## Evidence

- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types"
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [intro] we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices... we conduct the bimodality tests to judge whether a new: "we conduct the bimodality tests to judge whether a new"
- [readme] Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types.: "Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types."
