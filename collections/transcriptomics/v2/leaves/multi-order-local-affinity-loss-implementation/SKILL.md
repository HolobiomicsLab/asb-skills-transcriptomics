---
name: multi-order-local-affinity-loss-implementation
description: Use when fine-tuning a source-pretrained scRNA-seq annotation encoder
  on target data without source access and you need a self-supervised neighbor-based
  affinity loss on 'known' cells.
when_to_use_negative:
- Source data are still available — explicit feature alignment between source and
  target is possible, so source-free affinity regularization is unnecessary.
- The known/unknown partition of target cells has not yet been established — unknown
  neighbors must be excluded, so applying L_nbht without the adaptive manifold-mixup
  threshold risks noisy supervision.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Framework whose source-free model adaptation stage gathers multi-order neighborhood
    messages globally and imposes local affinity regularizations on 'known' cells;
    the L_nbht module is added to the ZINB denoising loss during encoder fine-tuning
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:6fd6a5245a73d4d09b5a35077224797921e3c21c345afc7c973effaf246904ef
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
yes_rate: 1.0
n_claims: 10
---

# multi-order-local-affinity-loss-implementation

## Summary

Implements scEMAIL's multi-order local affinity loss (L_nbht), which retrieves bidirectional, unidirectional, and extensional neighbors of each target cell from offline memory banks and imposes weighted affinity regularizations on 'known' target cells during source-free model adaptation, mitigating batch effect via reliable self-supervised neighbor information.

## When to use

Use when fine-tuning a source-pretrained scRNA-seq annotation encoder on target data without source access and you need a self-supervised neighbor-based affinity loss on 'known' cells.

## When NOT to use

- Source data are still available — explicit feature alignment between source and target is possible, so source-free affinity regularization is unnecessary.
- The known/unknown partition of target cells has not yet been established — unknown neighbors must be excluded, so applying L_nbht without the adaptive manifold-mixup threshold risks noisy supervision.

## Inputs

- Target-data representation bank R (Nt x d) of latent embeddings stored without gradient propagation
- Target-data soft label bank S (Nt x K) of predicted probability vectors stored without gradient propagation
- Index set D of 'known' target cells and the 'unknown' set Dt to exclude
- Hyperparameter M (number of nearest neighbors) and alpha (default 0.1, unidirectional neighbor weight)

## Outputs

- Scalar local affinity loss L_nbht (Equation 15) suitable for addition to the ZINB denoising loss during encoder fine-tuning
- Neighbor index sets V1 (bidirectional), V2 (unidirectional), and V3 (extensional) per target cell, consistent with Equations (12)-(14)

## How to apply

Build two offline memory banks from the target data without gradient propagation: a representation bank R (Nt x d) of latent embeddings and a soft label bank S (Nt x K) of predicted probability vectors. For each target cell z_i, retrieve its M-nearest neighbors by cosine similarity via top-M argmax over the representation bank, deliberately including the cell itself as a self-regularization neighbor. Partition the retrieved neighbors into bidirectional (mutual nearest) neighbors V1 and unidirectional neighbors V2 = VM \ V1, then form extensional neighbors V3 as the neighbors-of-neighbors l in V_M(j) for j in V_M(i). Compute L_nbht as the sum over cells in the known set D of p_i^t dot (S_j for j in V1∩D + alpha*S_k for k in V2∩D + S_l for l in V3∩D), normalized by |D|, with alpha = 0.1 default weight on unidirectional neighbors and bidirectional neighbors receiving the largest affinity. Exclude any neighbors falling in the 'unknown' set Dt from the affinity computation to prevent noisy supervision, and add the scalar loss to the ZINB denoising loss during encoder fine-tuning.

## Related tools

- **scEMAIL** (Framework whose source-free model adaptation stage gathers multi-order neighborhood messages globally and imposes local affinity regularizations on 'known' cells; the L_nbht module is added to the ZINB denoising loss during encoder fine-tuning) — https://github.com/aster-ww/scEMAIL

## Examples

```
loss = L_nbht(R, S, D, M)  # R: Nt x d embeddings, S: Nt x K soft labels, D: known-cell indices; add scalar to ZINB loss with alpha=0.1
```

## Evaluation signals

- The module reproduces the L_nbht objective of Equation (15) with default alpha = 0.1 on test inputs.
- Neighbor indices for V1, V2, and V3 are consistent with Equations (12)-(14) (mutual-nearest vs one-way vs neighbors-of-neighbors retrieval).
- The module returns a finite scalar loss value on test inputs (script_runs check plus field/value checks on outputs).
- No neighbor from the 'unknown' set Dt contributes to the affinity computation.
- The wrapper accepts banks R and S, index set D, and M, and returns a single scalar suitable for addition to the ZINB denoising loss.

## Limitations

- Neighbor gathering is cosine-similarity based over global scope; batch-effect-driven representation errors early in adaptation could mis-rank neighbors until the loss regularizes the encoder.
- Cells in the 'unknown' set contribute no supervision, so the loss only regularizes 'known' target cells; a poorly calibrated threshold shrinks the effective supervision set.

## Evidence

- [readme] We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells.: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on “known” cells."
- [readme] These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors.: "These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors."
- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope.: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope."
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [abstract] an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
