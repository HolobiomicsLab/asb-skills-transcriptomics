---
name: nearest-neighbor-retrieval-cosine-similarity
description: Use when adapting a source-free scRNA-seq annotation model (e.g., scEMAIL)
  and you need globally-scoped neighborhood structure — bidirectional (mutual nearest),
  unidirectional, and extensional (neighbors-of-neighbors) sets — to regularize 'known'
  target cells.
when_to_use_negative:
- Raw source data are available and explicit feature alignment between source and
  target is possible — scEMAIL's neighborhood scheme is designed for the source-free
  setting
- The target cells belong to the 'unknown' (novel cell type) group — their neighbors
  are deliberately excluded from the affinity computation to avoid noisy supervision
- No pre-trained source model / representation bank exists yet; neighbor retrieval
  operates on encoder embeddings, not raw count matrices
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Implements the cosine-similarity top-M neighbor retrieval over the representation
    bank and the multi-order (bidirectional/unidirectional/extensional) local affinity
    loss L_nbht during source-free model adaptation
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:5597ce0e00b22f1e2bb7e91046b6c1ab003a289ac48e2252e108641c9c260832
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# nearest-neighbor-retrieval-cosine-similarity

## Summary

Retrieves each target cell's M nearest neighbors in global scope via cosine similarity over an offline representation bank, as used by scEMAIL to gather bidirectional, unidirectional, and extensional neighbors for multi-order local affinity constraints during source-free model adaptation.

## When to use

Use when adapting a source-free scRNA-seq annotation model (e.g., scEMAIL) and you need globally-scoped neighborhood structure — bidirectional (mutual nearest), unidirectional, and extensional (neighbors-of-neighbors) sets — to regularize 'known' target cells.

## When NOT to use

- Raw source data are available and explicit feature alignment between source and target is possible — scEMAIL's neighborhood scheme is designed for the source-free setting
- The target cells belong to the 'unknown' (novel cell type) group — their neighbors are deliberately excluded from the affinity computation to avoid noisy supervision
- No pre-trained source model / representation bank exists yet; neighbor retrieval operates on encoder embeddings, not raw count matrices

## Inputs

- Target-data latent embedding matrix (representation bank R, Nt x d) computed by the pre-trained ZINB autoencoder encoder
- Soft label bank S (Nt x K) of predicted probability vectors stored without gradient propagation
- Index set D of 'known' target cells (from the adaptive manifold-mixup threshold partition)
- Number of neighbors M and unidirectional weight alpha (default 0.1)

## Outputs

- Per-cell neighbor index lists partitioned into V1 (bidirectional), V2 (unidirectional), and V3 (extensional) sets
- Scalar local affinity loss L_nbht (Equation 15) to be added to the ZINB denoising loss during encoder fine-tuning

## How to apply

Build two offline memory banks from the target data: a representation bank R (Nt x d) of latent embeddings and a soft label bank S (Nt x K) of predicted probability vectors, stored without gradient propagation. For each target cell z_i, compute cosine similarity against R and take a top-M argmax, deliberately including the cell itself as a self-regularization neighbor. Partition the retrieved M-neighbors into bidirectional (mutual nearest) neighbors V1 and unidirectional neighbors V2 = V_M \ V1, then form extensional neighbors V3 as the neighbors-of-neighbors l in V_M(j) for j in V_M(i). During adaptation, apply the local affinity loss L_nbht only on the 'known' set D: sum p_i^t dot (S_j for j in V1∩D + alpha*S_k for k in V2∩D + S_l for l in V3∩D), normalized by |D| with default alpha = 0.1 so bidirectional neighbors receive the largest affinity weight. Exclude neighbors falling in the 'unknown' set from the affinity computation to prevent noisy supervision.

## Related tools

- **scEMAIL** (Implements the cosine-similarity top-M neighbor retrieval over the representation bank and the multi-order (bidirectional/unidirectional/extensional) local affinity loss L_nbht during source-free model adaptation) — https://github.com/aster-ww/scEMAIL

## Examples

```
import torch; sims = X @ R.t(); V_M = sims.topk(M).indices  # cosine-similarity top-M neighbors, incl. self
```

## Evaluation signals

- The loss reproduces the L_nbht objective of Equation (15) with default alpha = 0.1
- Neighbor indices are consistent with Equations (12)-(14): mutual nearest pairs form V1, the remainder of the top-M forms V2, and neighbors-of-neighbors form V3
- The module returns a finite scalar loss value on test inputs (script_runs check plus field/value checks on outputs)
- Neighbors in the 'unknown' set are excluded from the affinity sum (verify no unknown-cell indices appear in the loss computation)
- The cell itself is present in its own retrieved top-M neighbor list (self-regularization)

## Limitations

- Neighbor quality depends on the pre-trained source encoder; batch effect between source and target before adaptation can corrupt cosine-similarity neighborhoods, which is why affinity constraints are combined with the ZINB denoising loss
- Neighbor supervision is only applied to 'known' cells; novel/unknown cell types receive no affinity regularization
- Weights are asymmetric by design (bidirectional > extensional > alpha=0.1 unidirectional); mis-specifying the partition or alpha changes the regularization strength

## Evidence

- [abstract] We gather multi-order neighborhood messages globally and impose local affinity regularizations on "known" cells.: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on "known" cells."
- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope. Based on the gathered multi-order neighborhood message, we impose various local affinity"
- [readme] These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors.: "These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors."
