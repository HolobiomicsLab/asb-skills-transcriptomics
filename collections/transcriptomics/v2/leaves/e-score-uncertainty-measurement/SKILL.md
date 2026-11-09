---
name: e-score-uncertainty-measurement
description: Use when annotating target scRNA-seq cells with a source-free transfer
  learning model and you need per-cell uncertainty scores to drive downstream novel
  cell-type detection (bimodality testing) and known/unknown partitioning.
when_to_use_negative:
- Source raw data are fully available and explicit feature alignment between source
  and target is feasible — standard transfer learning annotation without uncertainty-based
  novelty perception suffices
- Target data are known a priori to contain no novel cell types under a closed setting
  — the E-score/bimodality machinery is unnecessary and closed-set annotation can
  proceed directly
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Provides the expert ensemble system (entropy EN, confidence CF, consistency
    CS via main and auxiliary classifiers) that computes the E-score for each target
    cell
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:3f6056ef309b1380fdd1d75238ffe687910163f69e6a1f8e88db376041820280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - ItClust
  - scEMAIL
  - scSemiCluster
  - scmap
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# e-score-uncertainty-measurement

## Summary

Computes scEMAIL's expert ensemble score (E-score) that quantifies per-cell annotation uncertainty in target scRNA-seq data by combining three complementary indices — normalized entropy (EN), maximum confidence (CF), and auxiliary-classifier consistency (CS) — as the first step of the novel cell-type perception module.

## When to use

Use when annotating target scRNA-seq cells with a source-free transfer learning model and you need per-cell uncertainty scores to drive downstream novel cell-type detection (bimodality testing) and known/unknown partitioning.

## When NOT to use

- Source raw data are fully available and explicit feature alignment between source and target is feasible — standard transfer learning annotation without uncertainty-based novelty perception suffices
- Target data are known a priori to contain no novel cell types under a closed setting — the E-score/bimodality machinery is unnecessary and closed-set annotation can proceed directly

## Inputs

- Pre-trained scEMAIL source classification model weights (main classifier plus two auxiliary co-training classifiers sharing hidden representation h)
- Target scRNA-seq expression matrix (count data for ZINB-based autoencoder encoding)
- Number of known source cell types K (for entropy normalization)

## Outputs

- e_scores.csv with per-cell E-score uncertainty values for every target cell
- Per-cell expert-component values (entropy EN, confidence CF, consistency CS) usable for downstream bimodality testing

## How to apply

Train the scEMAIL source classification model on well-labeled source data with a ZINB-based autoencoder denoising loss plus supervised cross-entropy, and introduce two auxiliary classifiers c1 and c2 that take features from the shared hidden representation h. For each target cell, compute the E-score as an ensemble of three experts: entropy (EN) of the predictive distribution, confidence (CF) as the maximum predicted probability, and consistency (CS) between the main and auxiliary classifiers — consistency is robust to batch-effect-induced errors because both classifiers are unlikely to misclassify a cell into exactly the same type. Write per-cell E-score values to e_scores.csv, preserving the order of target cells. These scores feed the next perception step: bimodality tests (bimodality coefficient and Hartigan's dip test) on the empirical E-score distribution, which exhibits an apparent bimodal structure when target data contain novel cell types.

## Related tools

- **scEMAIL** (Provides the expert ensemble system (entropy EN, confidence CF, consistency CS via main and auxiliary classifiers) that computes the E-score for each target cell) — https://github.com/aster-ww/scEMAIL

## Examples

```
import pandas as pd; e_scores = pd.read_csv('e_scores.csv');  # per-cell EN/CF/CS ensemble uncertainty values for bimodality testing
```

## Evaluation signals

- e_scores.csv exists, is valid, and contains one E-score per target cell with no missing values
- Per-cell E-scores are bounded as normalized uncertainty values (entropy divided by log K, max confidence in [0,1], consistency a proportion)
- E-score distribution of a target dataset with known target-private cell types (e.g., open-setting 'mammary gland', open-partial 'neonatal rib') shows apparent bimodal structure, detectable by the bimodality coefficient or Hartigan's dip test
- Consistency component flags fewer false-uncertain cells than entropy under simulated batch effect, since both classifiers rarely misclassify into the same type
- Downstream adaptive threshold (average E-score over manifold-mixup pairs) partitions target cells such that the low-E-score group matches true target-private cell types

## Limitations

- E-score computation relies on a source model; batch effect between source and target can distort individual uncertainty components (mitigated, not eliminated, by the consistency expert)
- Interpretability of E-scores depends on the target containing the assumption's bimodal structure; unimodal E-score distributions in novel-type-free data must not be read as evidence of novel types
- Full implementation details are deferred to supplementary File S1 rather than the main text

## Evidence

- [methods] we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS).: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type.: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] First, an expert ensemble system measures uncertainty of each cell from three complementary aspects.: "First, an expert ensemble system measures uncertainty of each cell from three complementary aspects."
- [intro] we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices: "we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices"
