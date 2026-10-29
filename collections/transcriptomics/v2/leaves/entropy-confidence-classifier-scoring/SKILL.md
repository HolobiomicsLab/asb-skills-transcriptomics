---
name: entropy-confidence-classifier-scoring
description: Use when annotating scRNA-seq target cells with a pre-trained source
  classifier and you need a per-cell uncertainty score in [0,1] to detect novel cell
  types without source data.
when_to_use_negative:
- Source raw data are available and explicit feature alignment (e.g., Seurat v3 anchoring
  or scArches surgery) is preferred over a source-free scheme
- No pre-trained classifier outputs (main + two auxiliary probability vectors) exist
  yet — the scoring requires all three classifiers
- The goal is a final cell-type label rather than an uncertainty score; E-scores are
  an intermediate signal meant to feed bimodality tests and the manifold-mixup threshold,
  not a standalone annotation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Implements the expert ensemble system (entropy, confidence, consistency) that
    computes E-scores from the pre-trained source model's classifier outputs
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:7d5ea80ca9f4f77e24d08eae0cb9a6b250089543e57d100f0d4d3b844907fef1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# entropy-confidence-classifier-scoring

## Summary

Computes scEMAIL's per-cell E-score by combining three complementary uncertainty indices — entropy (EN), confidence (CF), and consistency (CS) — derived from a pre-trained source model's main and two auxiliary classifier outputs, enabling novel cell-type perception in scRNA-seq target data.

## When to use

Use when annotating scRNA-seq target cells with a pre-trained source classifier and you need a per-cell uncertainty score in [0,1] to detect novel cell types without source data.

## When NOT to use

- Source raw data are available and explicit feature alignment (e.g., Seurat v3 anchoring or scArches surgery) is preferred over a source-free scheme
- No pre-trained classifier outputs (main + two auxiliary probability vectors) exist yet — the scoring requires all three classifiers
- The goal is a final cell-type label rather than an uncertainty score; E-scores are an intermediate signal meant to feed bimodality tests and the manifold-mixup threshold, not a standalone annotation

## Inputs

- Latent representations z_it of target cells from the pre-trained source model's encoder
- K-dimensional soft probability vector p_i from the source model's main classifier c for each target cell
- Soft probability vectors p_it,1 and p_it,2 from the two auxiliary co-training classifiers cj (j=1,2)
- Number of known source cell types K (for entropy normalization)

## Outputs

- Per-cell E-scores in [0,1] written to e_scores.csv

## How to apply

Load the latent representations z_it and soft probability vectors from the source model's main classifier c and the two auxiliary classifiers cj (j=1,2). Compute normalized entropy EN(z_it) = -sum_k p_ik * log(p_ik) / log K from the K-dimensional main-classifier vector, confidence CF(z_it) = max_k p_ik from the same vector, and consistency CS(z_it) as the element-wise product of the two auxiliary classifiers' probability vectors p_it,1 · p_it,2. Average the three components as E-score(z_it) = (1 - EN(z_it)/log K + CF(z_it) + CS(z_it)) / 3, where higher scores indicate greater certainty that a cell belongs to a known type. Consistency is deliberately robust to batch-effect-induced errors because both auxiliary classifiers are unlikely to mispredict a cell into exactly the same type. Write the per-cell scores to e_scores.csv for downstream bimodality testing and adaptive-threshold partitioning.

## Related tools

- **scEMAIL** (Implements the expert ensemble system (entropy, confidence, consistency) that computes E-scores from the pre-trained source model's classifier outputs) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- e_scores.csv exists after execution
- Every E-score value falls within [0,1], consistent with the normalized components (EN/log K, CF, CS) defined in the methods
- Entropy component equals -sum_k p_ik * log(p_ik) / log K, i.e., lies in [0,1] for any valid probability vector
- When target data contain novel cell types, the empirical distribution of E-scores should exhibit an apparent bimodal structure, which downstream bimodality tests should detect

## Limitations

- E-scores require all three classifiers (one main and two auxiliary co-training classifiers) from the pre-trained source model; a model with only a single classifier cannot provide the consistency term
- The E-score is an intermediate uncertainty measure — detecting novel types still requires bimodality tests and an adaptive threshold via manifold mixup
- Implementation details are deferred to supplementary File S1, so exact hyperparameters may not be recoverable from the main text alone

## Evidence

- [methods] we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS): "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Consistency is robust to prediction errors caused by the batch effect: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [methods] their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] an expert ensemble system measures uncertainty of each cell from three complementary aspects: "First, an expert ensemble system measures uncertainty of each cell from three complementary aspects."
