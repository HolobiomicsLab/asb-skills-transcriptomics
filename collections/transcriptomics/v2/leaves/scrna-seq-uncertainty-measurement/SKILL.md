---
name: scrna-seq-uncertainty-measurement
description: Use when a pre-trained source classifier's soft probability outputs exist
  for scRNA-seq target cells and you need a per-cell uncertainty score to detect which
  cells belong to known cell types versus novel types.
when_to_use_negative:
- No pre-trained source model with a main classifier and two auxiliary classifiers
  is available — the E-score is defined only from those three probability outputs
- You already have confirmed cell-type labels for all target cells and only need final
  annotation, not uncertainty estimation or novel-type detection
- Downstream analysis requires distinguishing known from unknown cells without following
  through with bimodality testing and the manifold-mixup adaptive threshold, for which
  the E-score alone is only an intermediate signal
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Provides the pre-trained source model (main classifier c plus two auxiliary
    classifiers cj) whose classifier outputs are combined into the per-cell E-score;
    the expert ensemble system is part of scEMAIL's novel cell-type perception module
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:c7133550ddf1ba8c9af0d2510c67ce8159bcb6733e2e3d55af709a5f2ac860a4
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# scrna-seq-uncertainty-measurement

## Summary

Computes per-cell E-scores for scRNA-seq target data by combining three complementary uncertainty indices — entropy (EN), confidence (CF), and consistency (CS) — derived from a pre-trained source model's main and auxiliary classifier probability vectors, as implemented in scEMAIL.

## When to use

Use when a pre-trained source classifier's soft probability outputs exist for scRNA-seq target cells and you need a per-cell uncertainty score to detect which cells belong to known cell types versus novel types.

## When NOT to use

- No pre-trained source model with a main classifier and two auxiliary classifiers is available — the E-score is defined only from those three probability outputs
- You already have confirmed cell-type labels for all target cells and only need final annotation, not uncertainty estimation or novel-type detection
- Downstream analysis requires distinguishing known from unknown cells without following through with bimodality testing and the manifold-mixup adaptive threshold, for which the E-score alone is only an intermediate signal

## Inputs

- Latent representations z_it of target cells from the pre-trained scEMAIL source model autoencoder
- K-dimensional soft probability vector p_it from the source model's main classifier c for each target cell
- Soft probability vectors p_it,1 and p_it,2 from the two auxiliary co-training classifiers cj (j=1,2)
- Number of known source cell types K (for entropy normalization by log K)

## Outputs

- e_scores.csv containing a per-cell E-score in [0,1] for every target cell
- Per-component values (normalized entropy EN, confidence CF, consistency CS) feeding the ensemble average

## How to apply

Load the latent representations z_it and the K-dimensional soft probability vectors from the source model's main classifier c and the two auxiliary classifiers cj (j=1,2) for every target cell. Compute normalized entropy EN(z_it) = -sum_k p_ik * log(p_ik) divided by log K (so it lies in [0,1]), confidence CF(z_it) = max_k p_ik from the main classifier, and consistency CS(z_it) as the element-wise product of the two auxiliary classifiers' probability vectors p_it,1 · p_it,2. Average the three components: E-score(z_it) = (1 - EN(z_it)/log K + CF(z_it) + CS(z_it)) / 3, where a higher score means greater certainty the cell belongs to a known type. Consistency is the key guard against batch effect: it is unlikely that both auxiliary classifiers will wrongly predict a cell into exactly the same type, so CS remains robust where CF/EN may not. Write the per-cell E-scores to e_scores.csv for downstream bimodality testing and adaptive threshold partitioning.

## Related tools

- **scEMAIL** (Provides the pre-trained source model (main classifier c plus two auxiliary classifiers cj) whose classifier outputs are combined into the per-cell E-score; the expert ensemble system is part of scEMAIL's novel cell-type perception module) — https://github.com/aster-ww/scEMAIL

## Examples

```
import numpy as np; e_scores = ((1 - EN / np.log(K)) + CF + (p1 * p2).mean(axis=1)) / 3; np.savetxt('e_scores.csv', e_scores)
```

## Evaluation signals

- e_scores.csv exists and contains one E-score per target cell
- Every E-score value falls within the range [0,1], consistent with each normalized component (EN/log K, CF, CS) lying in [0,1]
- Normalized entropy values lie in [0,1] after division by log K
- E-score distribution is unimodal when the target data contain only known cell types, and shows an apparent bimodal structure when novel cell types are present
- Cells labeled as 'unknown' downstream have systematically lower E-scores than 'known' cells after the manifold-mixup adaptive threshold is applied

## Limitations

- Confidence and entropy from the main classifier are sensitive to batch-effect-induced prediction errors; only the consistency term is explicitly robust because both auxiliary classifiers are unlikely to mispredict a cell into the same type simultaneously
- The E-score alone does not decide novel-type presence — bimodality tests on the E-score distribution and an adaptive threshold via manifold mixup are required to partition known vs unknown cells
- Implementation details beyond Algorithm 1 are deferred to supplementary File S1, so exact numerics (e.g., aggregation of the element-wise consistency product) may require consulting the code

## Evidence

- [methods] Expert ensemble system for uncertainty measurement: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Consistency robustness rationale: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [intro] E-score purpose: "we first provide each target cell with an expert ensemble score (E-score) for uncertainty measurement combining three complementary indices"
- [methods] Bimodality signal in E-score distribution: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] README framing of the expert ensemble: "First, an expert ensemble system measures uncertainty of each cell from three complementary aspects."
