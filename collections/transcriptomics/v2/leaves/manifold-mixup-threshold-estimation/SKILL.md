---
name: manifold-mixup-threshold-estimation
description: Use when a source-free scRNA-seq annotation model's E-score distribution
  is judged bimodal (bimodality coefficient or Hartigan's dip test), and you must
  split target cells into known vs. unknown (novel-type) groups without a manually
  chosen cutoff.
when_to_use_negative:
- Bimodality tests indicate no novel cell types in the target data — then all cells
  are treated as known (D = all cells, Dt = empty) and no threshold is needed.
- Source data are available and a supervised or data-aligning method (e.g., Seurat
  v3, ItClust) is acceptable; scEMAIL and its perception module target the source-free
  setting where raw source data cannot be shared.
- Input is not single-cell data with a pre-trained scEMAIL-style source model providing
  latent vectors and auxiliary classifiers — the mixup pairs and E-scores have no
  defined inputs otherwise.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Provides the pre-trained ZINB autoencoder encoder, main and auxiliary classifiers,
    E-score computation, and the manifold-mixup adaptive threshold module that performs
    the known/unknown partition.
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:90c7aef9367b237dc421e44171d84223213f6ca6767a1b4c966f371e6eb61de1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - ItClust
  - Seurat v3
  - scArches
  - scEMAIL
  - scSemiCluster
  - scmap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# manifold-mixup-threshold-estimation

## Summary

scEMAIL's adaptive, data-based threshold procedure: once bimodality tests confirm novel cell types in target scRNA-seq data, synthetic manifold-mixup representations are built from all pairs of target latent vectors and their average expert-ensemble (E-score) sets the cutoff that partitions cells into 'known' and 'unknown' groups, replacing subjective user-defined thresholds.

## When to use

Use when a source-free scRNA-seq annotation model's E-score distribution is judged bimodal (bimodality coefficient or Hartigan's dip test), and you must split target cells into known vs. unknown (novel-type) groups without a manually chosen cutoff.

## When NOT to use

- Bimodality tests indicate no novel cell types in the target data — then all cells are treated as known (D = all cells, Dt = empty) and no threshold is needed.
- Source data are available and a supervised or data-aligning method (e.g., Seurat v3, ItClust) is acceptable; scEMAIL and its perception module target the source-free setting where raw source data cannot be shared.
- Input is not single-cell data with a pre-trained scEMAIL-style source model providing latent vectors and auxiliary classifiers — the mixup pairs and E-scores have no defined inputs otherwise.

## Inputs

- Per-cell expert ensemble E-scores for the target scRNA-seq dataset (from entropy EN/logK, max confidence CF, and auxiliary-classifier consistency CS)
- Target latent representations z_i from the ZINB autoencoder encoder h of the pre-trained scEMAIL source model
- Bimodality test statistics (bimodality coefficient and Hartigan's dip test) applied to the empirical E-score distribution

## Outputs

- Adaptive threshold δ (mean E-score over all manifold-mixup pairs, Eq. 10)
- Known set D (E-score > δ) and unknown set Dt (E-score ≤ δ) per-cell assignments for downstream annotation
- Per-cell 'unknown' labels carried into the final annotation output (e.g., scemail_crossmodality_annotations.csv)

## How to apply

After the expert ensemble E-score (average of normalized entropy EN/logK, maximum softmax confidence CF, and consistency CS from two auxiliary classifiers) is computed for each target cell, first confirm bimodality with both the bimodality coefficient and Hartigan's dip test; proceed only if at least one indicates bimodality. Then construct synthetic representations z~ = λ*z_i + (1−λ)*z_j for every pair of target latent vectors z_i, z_j with mixing coefficient λ drawn from Uniform(0,1) (manifold mixup). Set the adaptive threshold δ to the mean E-score over all possible pairs of target cells (Eq. 10 in the paper). Assign each target cell with E-score > δ to the known set D (later annotated by the classifier's argmax) and cells with E-score ≤ δ to the unknown set Dt (annotated as 'unknown'). Re-evaluate D and Dt each iteration of downstream model adaptation, since partitions can shift as the encoder fine-tunes; if no bimodality is detected, set D to all cells and leave Dt empty.

## Related tools

- **scEMAIL** (Provides the pre-trained ZINB autoencoder encoder, main and auxiliary classifiers, E-score computation, and the manifold-mixup adaptive threshold module that performs the known/unknown partition.) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- The threshold δ must equal the mean E-score computed over all pairs of target cells after manifold mixup (λ ~ Uniform(0,1)), as specified in Eq. 10.
- Ablation behavior should match the paper: in open and open-partial settings (e.g., mammary gland, neonatal rib), partitioning into known/unknown groups during adaptation increases classification accuracy relative to manual-threshold competitors (scmap, ItClust, scSemiCluster).
- Per-cell partition output must cover every target cell with exactly one assignment (D or Dt), and cells in Dt must carry the 'unknown' label in final annotations.
- Precision/recall of the unknown set Dt against true target-private cell types should show scEMAIL maintaining stable novel-type detection across settings while manually thresholded competitors degrade.
- Bimodality must be confirmed by at least one of the two tests (bimodality coefficient or Hartigan's dip test) before any threshold is applied; if neither fires, D should be all cells.

## Limitations

- The threshold is only meaningful when the empirical E-score distribution exhibits bimodal structure; the paper notes this bimodality appears specifically when target data contain novel cell types.
- The partition (D, Dt) is dynamic and must be refreshed every iteration during model adaptation, so a one-shot static partition may not reflect the converged assignment.
- Detailed implementation settings and some experimental details (e.g., simulated experiments, scalability checks) are deferred to supplementary File S1 and are not fully specified in the main text.

## Evidence

- [intro] target cells are then separated into "known" and "unknown" groups based on an adaptive data-based threshold by manifold mixup.: [evidence span withheld — anchor status: not_found]
- [methods] The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
