---
name: scrna-seq-memory-bank-management
description: Use when adapting a pre-trained source classifier to target scRNA-seq
  cells without source data and you need globally retrieved, weighted neighbors (bidirectional/unidirectional/extensional)
  as self-supervised affinity constraints.
when_to_use_negative:
- When raw source data are available, since explicit feature alignment methods (e.g.,
  transfer learning with source access) are then applicable and this source-free bank-based
  approach is unnecessary.
- When no pre-trained source model exists to produce soft label bank S and latent
  representation bank R — the banks depend on the source classifier's outputs.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: 'Implements the source-free adaptation pipeline: builds representation and
    soft label memory banks, retrieves multi-order neighbors, and optimizes L_nbht
    alongside the ZINB denoising loss.'
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:102686684fb1ad682cfa9105bbb49162f55e1b5bc5650ed8829f214017e8a191
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# scrna-seq-memory-bank-management

## Summary

Builds offline memory banks of latent representations and soft predicted labels from target scRNA-seq cells, enabling efficient global retrieval of multi-order neighbors that impose local affinity constraints during source-free model adaptation in scEMAIL.

## When to use

Use when adapting a pre-trained source classifier to target scRNA-seq cells without source data and you need globally retrieved, weighted neighbors (bidirectional/unidirectional/extensional) as self-supervised affinity constraints.

## When NOT to use

- When raw source data are available, since explicit feature alignment methods (e.g., transfer learning with source access) are then applicable and this source-free bank-based approach is unnecessary.
- When no pre-trained source model exists to produce soft label bank S and latent representation bank R — the banks depend on the source classifier's outputs.

## Inputs

- Target scRNA-seq cell latent representation matrix R (Nt x d)
- Soft predicted label matrix S (Nt x K) from the source classifier
- Index set D of 'known' target cells
- Hyperparameter M (number of nearest neighbors)
- Weight alpha (default 0.1) for unidirectional neighbors

## Outputs

- Scalar local affinity loss L_nbht for addition to the ZINB denoising loss
- Neighbor index sets V1 (bidirectional), V2 (unidirectional), V3 (extensional) per target cell

## How to apply

Construct two offline memory banks from the target data without gradient propagation: a representation bank R (Nt x d) of latent embeddings from the ZINB-based autoencoder and a soft label bank S (Nt x K) of predicted probability vectors. For each target cell z_i, retrieve its M-nearest neighbors via cosine similarity using a top-M argmax over R, deliberately including the cell itself as a self-regularization neighbor. Partition the retrieved neighbors into bidirectional (mutual nearest) neighbors V1, unidirectional neighbors V2 = V_M \ V1, and extensional neighbors V3 formed from neighbors-of-neighbors l in V_M(j) for j in V_M(i). Compute the local affinity loss L_nbht as a sum over cells in the 'known' set D of p_i^t dotted with the weighted combination of soft labels from V1, V2 (alpha = 0.1 default), and V3 neighbors, normalized by |D|, excluding any neighbors falling in the 'unknown' set to prevent noisy supervision. Return the scalar affinity loss given banks R, S, the known index set D, and M, suitable for addition to the ZINB denoising loss during encoder fine-tuning.

## Related tools

- **scEMAIL** (Implements the source-free adaptation pipeline: builds representation and soft label memory banks, retrieves multi-order neighbors, and optimizes L_nbht alongside the ZINB denoising loss.) — https://github.com/aster-ww/scEMAIL

## Examples

```
from scEMAIL.adapt import NeighborhoodBank; L_nbht = NeighborhoodBank(R, S, D, M=10, alpha=0.1).affinity_loss(p_t)
```

## Evaluation signals

- The returned loss reproduces the L_nbht objective of Equation (15) with default alpha = 0.1.
- Neighbor index sets are consistent with Equations (12)-(14) (bidirectional, unidirectional, extensional partitions).
- The loss returns a finite scalar value on test inputs (script_runs check plus field/value checks on outputs).
- Cells in the 'unknown' set Dt are excluded from the affinity computation, so no neighbor in Dt contributes to the loss.
- Bidirectional neighbors receive strictly larger affinity weights than unidirectional (alpha = 0.1) and extensional neighbors in the weighted sum.

## Limitations

- The quality of the affinity constraints depends on the source model's soft labels; wrong predictions by the source model propagate into bank S.
- Only 'known' cells contribute to the loss; if the novel-cell-type perception module misclassifies novel cells as known, noisy supervision enters the affinity objective.
- Details such as the exact value of M and other implementation specifics are deferred to File S1 rather than fully specified in the main text.

## Evidence

- [readme] We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells.: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells."
- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope.: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope."
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [readme] an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
