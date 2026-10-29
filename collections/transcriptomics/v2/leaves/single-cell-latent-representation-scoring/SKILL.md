---
name: single-cell-latent-representation-scoring
description: Use when a source-free transfer learning pipeline needs per-cell uncertainty
  for target scRNA-seq cells from classifier outputs, e.g. to feed bimodality tests
  for novel cell-type detection in scEMAIL.
when_to_use_negative:
- The source model has not been trained yet — the score depends on the main classifier
  plus the two auxiliary classifiers taking features from h of the pre-trained source
  model
- You need hard cell-type labels rather than an uncertainty signal — E-scores only
  measure certainty and must be followed by bimodality tests and a manifold-mixup
  threshold to actually partition cells
- Raw source data are available and you can align features explicitly — the source-free
  E-score design targets the setting where source data are inaccessible during adaptation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Implements the expert ensemble system (entropy, confidence, consistency) that
    computes the per-cell E-score from source-classifier outputs on target latent
    representations
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:3bcfb690da8dfc9937c2973963ceb8e83e4db7c074eacdef649dabbfda756445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-latent-representation-scoring

## Summary

Computes a per-cell expert-ensemble E-score from the soft probability vectors of a pre-trained source model's main classifier and two auxiliary classifiers over target-cell latent representations (z_it), combining normalized entropy (EN), confidence (CF), and consistency (CS) to quantify how certain each target cell belongs to a known cell type.

## When to use

Use when a source-free transfer learning pipeline needs per-cell uncertainty for target scRNA-seq cells from classifier outputs, e.g. to feed bimodality tests for novel cell-type detection in scEMAIL.

## When NOT to use

- The source model has not been trained yet — the score depends on the main classifier plus the two auxiliary classifiers taking features from h of the pre-trained source model
- You need hard cell-type labels rather than an uncertainty signal — E-scores only measure certainty and must be followed by bimodality tests and a manifold-mixup threshold to actually partition cells
- Raw source data are available and you can align features explicitly — the source-free E-score design targets the setting where source data are inaccessible during adaptation

## Inputs

- Latent representations z_it of target cells from the pre-trained source model encoder
- K-dimensional soft probability vector from the source model's main classifier c for each target cell
- Soft probability vectors from the two auxiliary co-training classifiers cj (j=1,2) for each target cell
- Number of source cell types K (for entropy normalization by log K)

## Outputs

- e_scores.csv with one E-score per target cell, each value in [0,1]

## How to apply

Load the latent representations z_it and the soft probability vectors produced by the pre-trained source model's main classifier c and two auxiliary classifiers cj (j=1,2) for every target cell. Compute entropy EN(z_it) = -sum_k p_ik * log(p_ik) from the K-dimensional main-classifier vector and normalize by log K so it lies in [0,1]; compute confidence CF(z_it) = max_k p_ik; compute consistency CS(z_it) as the element-wise product p_it,1 · p_it,2 of the two auxiliary classifiers' probability vectors. Average the three components as E-score(z_it) = (1 - EN(z_it)/log K + CF(z_it) + CS(z_it)) / 3, where a higher score means greater certainty the cell is a known type. Consistency is deliberately used because it is robust to batch-effect-induced prediction errors — it is unlikely both auxiliary classifiers wrongly predict the same cell type. Write per-cell scores to e_scores.csv; these scores later drive bimodality tests and the manifold-mixup adaptive threshold for partitioning cells into 'known' and 'unknown' groups.

## Related tools

- **scEMAIL** (Implements the expert ensemble system (entropy, confidence, consistency) that computes the per-cell E-score from source-classifier outputs on target latent representations) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- e_scores.csv exists and contains one score per target cell
- Every E-score value falls within [0,1], consistent with the normalized components (EN/log K, CF, CS)
- Entropy component is normalized by log K, so it lies within [0,1] before averaging
- When target data contain novel cell types, the empirical distribution of E-scores should exhibit an apparent bimodal structure, enabling downstream bimodality tests
- CS component equals the element-wise product p_it,1 · p_it,2 of the two auxiliary classifiers' probability vectors

## Limitations

- Implementation details are deferred to File S1 and Algorithm 1, not fully given in the main methods text
- The score alone does not decide novelty: bimodality tests must confirm novel cell types before the adaptive manifold-mixup threshold partitions cells into known/unknown groups
- Consistency mitigates but does not eliminate batch-effect-induced prediction errors from the source model

## Evidence

- [methods] we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS).: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type.: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [intro] we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices: "we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices"
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] First, an expert ensemble system measures uncertainty of each cell from three complementary aspects.: "First, an expert ensemble system measures uncertainty of each cell from three complementary aspects."
