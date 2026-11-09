---
name: auxiliary-classifier-ensemble-consistency
description: Use when annotating target scRNA-seq cells source-free with scEMAIL and
  you need per-cell uncertainty from the main classifier plus two auxiliary classifiers
  to later detect novel cell types via E-score bimodality.
when_to_use_negative:
- Source raw data are available, so uncertainty can be derived directly from supervised
  alignment rather than the source-free expert ensemble
- You only need a final cell-type label with no novel-cell-type perception — a plain
  classifier argmax suffices without E-score computation
- The pre-trained source model lacks the two auxiliary co-training classifiers, so
  CS cannot be computed
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Framework whose pre-trained main classifier c and two auxiliary classifiers
    cj supply the entropy, confidence, and consistency components combined into the
    E-score
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:b7a16d74ce0e2ed33aa8d20cb28ca938a6c87943337749213b50061aacbdb547
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# auxiliary-classifier-ensemble-consistency

## Summary

Computes a per-cell expert-ensemble E-score for scRNA-seq target cells by combining normalized entropy, confidence, and auxiliary-classifier consistency from a pre-trained source model, quantifying how certain each cell belongs to a known cell type in scEMAIL's novel cell-type perception module.

## When to use

Use when annotating target scRNA-seq cells source-free with scEMAIL and you need per-cell uncertainty from the main classifier plus two auxiliary classifiers to later detect novel cell types via E-score bimodality.

## When NOT to use

- Source raw data are available, so uncertainty can be derived directly from supervised alignment rather than the source-free expert ensemble
- You only need a final cell-type label with no novel-cell-type perception — a plain classifier argmax suffices without E-score computation
- The pre-trained source model lacks the two auxiliary co-training classifiers, so CS cannot be computed

## Inputs

- Latent representations z_it from the pre-trained scEMAIL source autoencoder for every target cell
- K-dimensional soft probability vectors from the main classifier c of the pre-trained source model
- Probability vectors from the two auxiliary classifiers cj (j=1,2) computed from the shared feature representation h

## Outputs

- Per-cell expert ensemble E-scores (values in [0,1]) written to e_scores.csv
- E-score empirical distribution used downstream for bimodality testing and adaptive (manifold mixup) known/unknown thresholding

## How to apply

For each target cell, take the K-dimensional soft probability vector from the pre-trained source model's main classifier c and the probability vectors from the two auxiliary co-training classifiers cj (j=1,2) applied to latent features h. Compute entropy EN = -sum_k p_ik * log(p_ik) normalized by log K to [0,1], confidence CF = max_k p_ik, and consistency CS as the element-wise product of the two auxiliary probability vectors p_it,1 · p_it,2. Average the three components as E-score(z_it) = (1 - EN/log K + CF + CS) / 3, so higher scores indicate greater certainty of a known cell type. Consistency is robust to batch-effect-induced errors because both auxiliary classifiers rarely mispredict a cell into exactly the same type. Write per-cell scores to e_scores.csv and check every value lies in [0,1] before running downstream bimodality tests and manifold-mixup adaptive thresholding.

## Related tools

- **scEMAIL** (Framework whose pre-trained main classifier c and two auxiliary classifiers cj supply the entropy, confidence, and consistency components combined into the E-score) — https://github.com/aster-ww/scEMAIL

## Examples

```
import numpy as np, pandas as pd; e = (1 - EN/np.log(K) + CF + CS)/3; pd.DataFrame({'e_score': e}).to_csv('e_scores.csv', index=False)
```

## Evaluation signals

- e_scores.csv exists and every E-score value falls within [0,1], consistent with the log K-normalized entropy, max-probability confidence, and element-wise consistency components
- For cells confidently assigned by the main classifier (CF near 1 and EN near 0), the E-score should be near 1
- When the target data contain novel cell types, the empirical E-score distribution should exhibit apparent bimodal structure, confirming the ensemble is discriminating known vs. unknown cells
- Spot-check the CS term: it should remain high for cells both auxiliary classifiers agree on, and be robust to batch-effect-induced single-classifier mispredictions

## Limitations

- Novel cell types in the target data push E-scores bimodally, so the raw E-score alone is not a label; downstream bimodality tests and a manifold-mixup adaptive threshold are required to partition known vs. unknown groups
- Implementation details are deferred to File S1, so exact engineering choices beyond the published formula are not documented in the main text

## Evidence

- [methods] Expert ensemble system for uncertainty measurement: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Consistency robustness to batch effect: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [methods] Bimodal E-score distribution with novel cell types: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [methods] Auxiliary classifiers take shared features for uncertainty measurement: "we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement"
- [readme] E-score combining three complementary indices: "an expert ensemble system measures uncertainty of each cell from three complementary aspects."
