---
name: source-free-transfer-learning-model-adaptation
description: Use when labeled source scRNA-seq data are unavailable (privacy/storage
  constraints) but pre-trained source model weights exist, and target cells must be
  annotated including detection of novel cell types.
when_to_use_negative:
- Raw source data are available and explicit feature alignment (e.g., scArches-style
  architectural surgery on source and target) is feasible and preferred
- All target cell types are known a priori and a subjective/manual threshold for novel-type
  detection is acceptable with simpler transfer tools (scmap, Seurat v3, ItClust)
- Input is not single-cell omics data with a trained source classifier (the ZINB autoencoder
  and co-training classifiers presuppose scRNA-seq-like count structure)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: 'Implements the full source-free adaptation pipeline: ZINB autoencoder training,
    expert ensemble E-score, bimodality tests, manifold-mixup threshold, and multi-order
    local affinity fine-tuning'
  repo: https://github.com/aster-ww/scEMAIL
- name: scArches
  role: 'Contrasting baseline: transfer learning via single-cell architectural surgery
    that maps target data onto source data but requires source data access, unlike
    this source-free skill'
- name: ItClust
  role: 'Contrasting baseline: iterative deep transfer learning initialized from well-annotated
    source data; requires raw source data'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:9e11bf35e21e0e4c61834e2f22492dd078e0e78e870448a66627fee55c4a22ac
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# source-free-transfer-learning-model-adaptation

## Summary

Adapts a pre-trained single-cell annotation model to a target scRNA-seq dataset using only shared pre-trained model weights (no raw source data), combining a ZINB denoising loss with multi-order local affinity constraints to mitigate batch effect and annotate novel cell types.

## When to use

Use when labeled source scRNA-seq data are unavailable (privacy/storage constraints) but pre-trained source model weights exist, and target cells must be annotated including detection of novel cell types.

## When NOT to use

- Raw source data are available and explicit feature alignment (e.g., scArches-style architectural surgery on source and target) is feasible and preferred
- All target cell types are known a priori and a subjective/manual threshold for novel-type detection is acceptable with simpler transfer tools (scmap, Seurat v3, ItClust)
- Input is not single-cell omics data with a trained source classifier (the ZINB autoencoder and co-training classifiers presuppose scRNA-seq-like count structure)

## Inputs

- Pre-trained source model weights (ZINB autoencoder encoder + supervised classifier + two auxiliary classifiers)
- Target scRNA-seq count matrix (e.g., bone marrow, pancreas, mammary gland, neonatal rib dataset pairs)
- Optional: target scATAC-seq or CITE-seq data for cross-modality adaptation

## Outputs

- Per-target-cell annotation labels (argmax of classifier for known cells)
- 'unknown' labels for cells partitioned into the Dt group by the adaptive threshold
- Adapted encoder weights fine-tuned on target data
- E-score distribution and bimodality test results (bimodality coefficient, Hartigan's dip test)

## How to apply

Pre-train the source model with a ZINB model-based autoencoder (negative log-likelihood reconstruction loss), a supervised cross-entropy classifier, and two auxiliary co-training classifiers with different random initializations (Eq. 5); share only these weights. For each target cell compute the expert ensemble E-score as the average of normalized entropy (EN/logK), maximum confidence, and consistency (dot product of the two auxiliary classifiers' soft probability vectors; Eq. 8). Test the E-score distribution for bimodality with the bimodality coefficient and Hartigan's dip test; if either indicates bimodality, build manifold-mixup representations z~ = λ*z_i + (1-λ)*z_j with λ ~ Uniform(0,1) and set the adaptive threshold δ to the mean E-score over all target cell pairs (Eq. 10), splitting cells into known (D) and unknown (Dt) groups. Fine-tune only the encoder (classifier weights frozen) on target data with the ZINB denoising loss plus a multi-order local affinity loss over bidirectional, unidirectional (α = 0.1), and extensional neighbors restricted to known cells D, updating D and Dt each iteration (Eq. 17). Finally assign argmax classifier labels to D and 'unknown' to Dt. The rationale is that neighborhood-based self-supervised constraints compensate for the impossibility of explicit source-target feature alignment when raw source data are withheld.

## Related tools

- **scEMAIL** (Implements the full source-free adaptation pipeline: ZINB autoencoder training, expert ensemble E-score, bimodality tests, manifold-mixup threshold, and multi-order local affinity fine-tuning) — https://github.com/aster-ww/scEMAIL
- **scArches** (Contrasting baseline: transfer learning via single-cell architectural surgery that maps target data onto source data but requires source data access, unlike this source-free skill)
- **ItClust** (Contrasting baseline: iterative deep transfer learning initialized from well-annotated source data; requires raw source data)

## Examples

```
git clone https://github.com/aster-ww/scEMAIL && python run_scEMAIL.py --source_model checkpoints/bone_marrow_source.pt --target data/trachea_target.h5ad --setting open
```

## Evaluation signals

- Total annotation accuracy exceeds 0.7 on real scRNA-seq dataset pairs and matches reported Table 1 reference values (e.g., bone marrow 0.8873, trachea 0.9415) within tolerance
- scEMAIL ranks highest against baselines (scmap, Seurat v3, ItClust, scSemiCluster, scArches, source-only) across closed, partial, open, and open-partial settings
- E-score distribution shows bimodal structure (bimodality coefficient or Hartigan's dip test positive) exactly for target datasets containing novel cell types
- Known/unknown partition sums to the full target set, with Dt cells labeled 'unknown' and D cells receiving argmax classifier labels
- Fine-tuning updates only the encoder; classifier weights remain frozen throughout model adaptation

## Limitations

- Implementation details of competing methods and simulation/scalability experiments are deferred to File S1 and Table S1, not fully specified in the main text
- Cross-tissue task details are only in Table S5, and scATAC-seq 'mouse bone marrow' accuracies appear only graphically in Figure 10 without numerical values in text
- Batch effect can still cause low annotation accuracy on some dataset pairs (e.g., 'placenta' accuracy and 'peripheral blood' H-score were reported as rather low)

## Evidence

- [abstract] Source-free adaptation without raw source data: "We propose a universal annotation framework for scRNA-seq data called scEMAIL, which automatically detects novel cell types without accessing source data during adaptation."
- [methods] Expert ensemble E-score components: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Bimodality decision rule: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] Adaptive manifold-mixup threshold: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [results] Benchmark reference values: "Bone marrow0.85510.50930.85640.84980.84360.83880.8873"
- [readme] Local affinity regularization rationale: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells."
