---
name: neighborhood-regularization-in-transfer-learning
description: Use when adapting a pre-trained source classifier to unlabeled scRNA-seq
  target data source-free, and you need self-supervised regularization on 'known'
  cells using neighborhood structure to counter batch effect.
when_to_use_negative:
- When raw source data are available and explicit source-target feature alignment
  can be performed directly (scEMAIL is designed for the source-free setting)
- When the target cell has been assigned to the 'unknown' group — affinity constraints
  are imposed only on 'known' cells to avoid noisy supervision
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Implements the multi-order neighbor search (cosine similarity over offline
    memory banks) and the local affinity constraint loss added to the ZINB denoising
    loss during source-free model adaptation
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:bf43e48ca7fa02defcd9420ad87817a5aa8075db6f4ccf8862df8f4dfd61a7e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# neighborhood-regularization-in-transfer-learning

## Summary

A source-free transfer-learning regularization technique from scEMAIL that gathers multi-order neighbors (bidirectional, unidirectional, and extensional) of each target cell in global scope and imposes weighted local affinity constraints on 'known' cells, mitigating batch effect without accessing raw source data.

## When to use

Use when adapting a pre-trained source classifier to unlabeled scRNA-seq target data source-free, and you need self-supervised regularization on 'known' cells using neighborhood structure to counter batch effect.

## When NOT to use

- When raw source data are available and explicit source-target feature alignment can be performed directly (scEMAIL is designed for the source-free setting)
- When the target cell has been assigned to the 'unknown' group — affinity constraints are imposed only on 'known' cells to avoid noisy supervision

## Inputs

- Target scRNA-seq latent embedding matrix R (Nt x d) from the ZINB autoencoder encoder
- Soft label bank S (Nt x K) of predicted probability vectors from the adapted source classifier
- Index set D of 'known' target cells (from E-score threshold partitioning via manifold mixup)
- Neighborhood size parameter M and unidirectional weight alpha (default 0.1)

## Outputs

- Scalar local affinity loss L_nbht (Equation 15) addable to the ZINB denoising loss
- Neighbor index lists per target cell partitioned into V1 (bidirectional), V2 (unidirectional), V3 (extensional) per Equations (12)-(14)

## How to apply

Build two offline memory banks from target data: a representation bank R (Nt x d) of latent embeddings and a soft label bank S (Nt x K) of predicted probabilities, stored without gradient propagation. For each target cell z_i, retrieve its M-nearest neighbors by cosine similarity via top-M argmax over R, deliberately including the cell itself as a self-regularization neighbor. Partition neighbors into bidirectional (mutual nearest) V1, unidirectional V2 = V_M \ V1, and extensional V3 (neighbors-of-neighbors), then compute the local affinity loss L_nbht: sum over cells in the known set D of p_i^t dot (S_j for j in V1∩D + alpha*S_k for k in V2∩D + S_l for l in V3∩D), normalized by |D| with default alpha = 0.1 weighting unidirectional neighbors so bidirectional neighbors receive the largest affinity. Exclude neighbors in the 'unknown' set D_t from the affinity computation to avoid noisy supervision, and add the scalar L_nbht to the ZINB denoising loss during encoder fine-tuning.

## Related tools

- **scEMAIL** (Implements the multi-order neighbor search (cosine similarity over offline memory banks) and the local affinity constraint loss added to the ZINB denoising loss during source-free model adaptation) — https://github.com/aster-ww/scEMAIL

## Examples

```
loss_nbht = L_nbht(R, S, D, M)  # scalar local affinity loss added to ZINB denoising loss during encoder fine-tuning
```

## Evaluation signals

- The module reproduces the L_nbht objective of Equation (15) with default alpha = 0.1
- Retrieved neighbor indices remain consistent with the bidirectional/unidirectional/extensional definitions of Equations (12)-(14)
- The loss returns a finite scalar value on test inputs (script_runs check plus field/value checks)
- Bidirectional neighbors receive larger affinity weight than unidirectional (alpha = 0.1) and extensional neighbors
- No neighbor from the 'unknown' set D_t contributes to the affinity computation

## Limitations

- The regularization is only applied to 'known' cells; cells partitioned as 'unknown' by the adaptive manifold-mixup threshold receive no affinity constraint, so quality depends on the preceding E-score partitioning
- Implementation details are deferred to File S1, so exact hyperparameters beyond alpha = 0.1 may require consulting supplementary material

## Evidence

- [readme] We gather multi-order neighborhood messages globally and impose local affinity regularizations on "known" cells.: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on "known" cells."
- [readme] These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors.: "These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors."
- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope.: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope."
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [methods] The entire workflow of scEMAIL is displayed in Algorithm 1 and more implementation details can be seen in File S1.: "The entire workflow of scEMAIL is displayed in Algorithm 1 and more implementation details can be seen in File S1."
