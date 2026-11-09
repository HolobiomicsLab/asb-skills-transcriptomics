---
name: mutual-neighbor-graph-construction
description: Use when adapting a source-trained single-cell classifier to unlabeled
  target cells without raw source data and you need reliable self-supervised neighborhood
  structure for affinity regularization.
when_to_use_negative:
- Raw source data are available, so explicit source-target feature alignment (e.g.,
  scArches-style surgery or direct joint training) can be used instead of source-free
  neighbor-based regularization
- No pre-trained source model exists, so soft label bank S from predicted probabilities
  cannot be constructed
- The goal is de novo clustering of target data rather than regularizing annotation
  of an already source-trained classifier
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scEMAIL
  role: Implements the M-nearest-neighbor retrieval over the representation bank,
    the V1/V2/V3 multi-order partition, and the L_nbht local affinity loss applied
    to 'known' cells during model adaptation
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:90ee3c01bce3983e864a5752c8cad29a6042577d7b831e4cb4aa0d8f6f49e546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Mutual-neighbor graph construction

## Summary

Constructs multi-order neighbor sets (bidirectional, unidirectional, and extensional) for each scRNA-seq target cell by cosine-similarity top-M retrieval over an offline representation bank, as used by scEMAIL to impose local affinity constraints during source-free model adaptation.

## When to use

Use when adapting a source-trained single-cell classifier to unlabeled target cells without raw source data and you need reliable self-supervised neighborhood structure for affinity regularization.

## When NOT to use

- Raw source data are available, so explicit source-target feature alignment (e.g., scArches-style surgery or direct joint training) can be used instead of source-free neighbor-based regularization
- No pre-trained source model exists, so soft label bank S from predicted probabilities cannot be constructed
- The goal is de novo clustering of target data rather than regularizing annotation of an already source-trained classifier

## Inputs

- Target scRNA-seq latent representation bank R (Nt x d, no-gradient)
- Soft label bank S (Nt x K predicted probability vectors, no-gradient)
- Index set D of 'known' target cells (excluding 'unknown' group cells)
- Number of nearest neighbors M; unidirectional-neighbor weight alpha (default 0.1)

## Outputs

- Per-cell neighbor partition: bidirectional set V1, unidirectional set V2, extensional set V3
- Scalar local affinity loss L_nbht suitable for addition to the ZINB denoising loss during encoder fine-tuning

## How to apply

Build two offline memory banks from the target scRNA-seq data: a representation bank R (Nt x d) of latent embeddings and a soft label bank S (Nt x K) of predicted probabilities, both stored without gradient propagation. For each target cell z_i, retrieve its M-nearest neighbors by cosine similarity via a top-M argmax over R, deliberately including the cell itself as a self-regularization neighbor. Partition the retrieved set V_M into bidirectional (mutual nearest) neighbors V1 and unidirectional neighbors V2 = V_M \ V1, then form extensional neighbors V3 as the neighbors-of-neighbors appearing in V_M(j) for j in V_M(i). Feed these sets into the local affinity loss L_nbht, weighting bidirectional neighbors fully, unidirectional neighbors with alpha = 0.1 (default), and extensional neighbors fully; exclude any neighbors falling in the 'unknown' set to avoid noisy supervision. The rationale is that batch-effect-robust semantic relations among global neighbors supply reliable self-supervised signals that mitigate wrong source-model classifications during encoder fine-tuning alongside the ZINB denoising loss.

## Related tools

- **scEMAIL** (Implements the M-nearest-neighbor retrieval over the representation bank, the V1/V2/V3 multi-order partition, and the L_nbht local affinity loss applied to 'known' cells during model adaptation) — https://github.com/aster-ww/scEMAIL

## Examples

```
import numpy as np; sims = R @ R.T; V_M = np.argpartition(-sims, M, axis=1)[:, :M]; V1 = np.array([j for j in V_M[i] if i in V_M[j]])
```

## Evaluation signals

- The loss reproduces the L_nbht objective of Equation (15) with default alpha = 0.1
- Neighbor indices are consistent with the definitions in Equations (12)-(14): V1 are mutual nearest neighbors, V2 = V_M \ V1, and V3 contains neighbors-of-neighbors
- The affinity computation excludes all cells in the 'unknown' set Dt
- Bidirectional neighbors receive the largest affinity weight relative to unidirectional (alpha = 0.1) and extensional neighbors
- The module returns a finite scalar loss value on test inputs (script_runs check plus field/value checks on outputs)

## Limitations

- Neighbors falling in the 'unknown' set must be excluded; including them would inject noisy supervision from cells flagged as potential novel cell types
- Quality depends on the source-model soft labels in bank S; prediction errors caused by batch effect can corrupt neighbor semantics, which is why consistency-based uncertainty and 'known'/'unknown' partitioning precede this step
- Performance depends on choice of M and the alpha = 0.1 default weighting for unidirectional neighbors

## Evidence

- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope."
- [abstract] gather multi-order neighborhood messages and impose local affinity regularizations: "We gather multi-order neighborhood messages globally and impose local affinity regularizations on "known" cells."
- [readme] self-supervised neighbor information mitigates wrong classifications: "These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors."
