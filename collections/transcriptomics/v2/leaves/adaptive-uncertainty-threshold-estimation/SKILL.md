---
name: adaptive-uncertainty-threshold-estimation
description: Use when bimodality testing (bimodality coefficient, Hartigan's dip test)
  has confirmed novel cell types in target scRNA-seq data and a non-manual, data-driven
  cutoff is needed to separate known from unknown cells.
when_to_use_negative:
- When neither the bimodality coefficient nor Hartigan's dip test indicates a bimodal
  E-score distribution — then all target cells are defined as the known group and
  thresholding is skipped
- When raw source data are available and explicit source-target feature alignment
  is intended (scEMAIL is designed for source-free adaptation)
- When the user has already supplied a validated, externally justified cutoff for
  novel-cell detection
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: 'Implements the novel cell-type perception module: E-score computation, bimodality
    testing, and adaptive threshold via manifold mixup that partitions target cells
    into known and unknown groups'
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:e842691d6a05d6e7e30d511130636d5b7f0b7e2b343e9a1246b2b1ab6186f7dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# adaptive-uncertainty-threshold-estimation

## Summary

Derives a data-based threshold (delta) for partitioning scRNA-seq target cells into 'known' vs 'unknown' groups by computing expert-ensemble E-scores of manifold-mixup synthetic pairs, eliminating user-selected thresholds when novel cell types are present in scEMAIL.

## When to use

Use when bimodality testing (bimodality coefficient, Hartigan's dip test) has confirmed novel cell types in target scRNA-seq data and a non-manual, data-driven cutoff is needed to separate known from unknown cells.

## When NOT to use

- When neither the bimodality coefficient nor Hartigan's dip test indicates a bimodal E-score distribution — then all target cells are defined as the known group and thresholding is skipped
- When raw source data are available and explicit source-target feature alignment is intended (scEMAIL is designed for source-free adaptation)
- When the user has already supplied a validated, externally justified cutoff for novel-cell detection

## Inputs

- Target-cell latent representations from the scEMAIL ZINB-based autoencoder
- Per-cell expert-ensemble E-scores (entropy, confidence, consistency) from the source model and its two auxiliary classifiers
- Bimodality test results (bimodality coefficient and/or Hartigan's dip test) confirming novel cell types

## Outputs

- Adaptive threshold value delta (average E-score over all synthetic mixed cell pairs)
- Per-cell binary partition assigning each of the Nt target cells to the known group D (E-score > delta) or unknown group D_t

## How to apply

First, load target-cell latent representations and their expert-ensemble E-scores combining entropy (EN), confidence (CF), and consistency (CS) from the source model. If bimodality is confirmed by the bimodality coefficient or Hartigan's dip test, construct synthetic manifold representations by mixing every pair of distinct target cells z_i and z_j with coefficient lambda drawn from Uniform(0,1), per the manifold-mixup scheme. Compute the E-score of each synthetic mixed representation and set the threshold delta to the average E-score over all possible cell pairs. Partition targets by the rule: cell i is 'known' (group D) if E-score(z_i) > delta, otherwise it belongs to the unknown group D_t. The rationale is that mixed synthetic cells occupy the manifold between known and novel populations, so their mean uncertainty provides an objective decision boundary that avoids error propagation or overconfident predictions in subsequent network training.

## Related tools

- **scEMAIL** (Implements the novel cell-type perception module: E-score computation, bimodality testing, and adaptive threshold via manifold mixup that partitions target cells into known and unknown groups) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- Number of known cells plus number of unknown cells equals Nt (the total target-cell count)
- Every target cell with E-score > delta is labeled known, exactly matching the paper's partition rule (no known cell has E-score <= delta)
- Threshold delta equals the average E-score across all synthetic manifold-mixup pairs with lambda ~ Uniform(0,1)
- Bimodality of the E-score distribution was confirmed by at least one of the bimodality coefficient or Hartigan's dip test before thresholding was applied
- Cells assigned to the known group are subsequently used for local affinity regularization during model adaptation without error propagation

## Limitations

- If neither bimodality measure indicates a bimodal E-score distribution, the threshold is not computed and all target cells are treated as known
- Bimodal-structure detection and thresholding depend on the E-score quality from the source model's two auxiliary co-training classifiers
- Detailed implementation specifics are deferred to supplementary File S1 rather than fully given in the main text

## Evidence

- [methods] Adaptive Data-Based Threshold By Manifold Mixup: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [methods] Partition rule: [evidence span withheld — anchor status: not_found]
- [methods] Bimodality gating: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [readme] Threshold avoids user selection: "once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
- [readme] Rationale for known-cell constraints: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells."
