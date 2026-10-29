---
name: scrna-seq-latent-representation-scoring
description: Use when target scRNA-seq latent representations have confirmed bimodal
  E-score distributions (via bimodality coefficient or Hartigan's dip test) and cells
  must be split into known vs unknown groups without a manual threshold.
when_to_use_negative:
- Bimodality tests (bimodality coefficient, Hartigan's dip test) find no bimodal structure
  — then scEMAIL defines the known group as all target cells and skips thresholding
- Raw source data are available and can be aligned with target data — the source-free,
  threshold-free design is not needed
- A priori known, manually specified thresholds are acceptable and user supervision
  is available
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Framework providing the expert ensemble E-score, manifold-mixup threshold
    computation, and known/unknown partition of target cells
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:b65fc53ec4665d3105e1e813f04eb0a1f89dfb1b10f48add40de2b6aef8601d3
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

# scrna-seq-latent-representation-scoring

## Summary

Score target-cell latent representations in scEMAIL with an expert ensemble E-score (entropy, confidence, consistency) and partition cells into 'known' and 'unknown' groups using an adaptive manifold-mixup threshold, enabling automatic novel cell-type perception without a user-selected cutoff.

## When to use

Use when target scRNA-seq latent representations have confirmed bimodal E-score distributions (via bimodality coefficient or Hartigan's dip test) and cells must be split into known vs unknown groups without a manual threshold.

## When NOT to use

- Bimodality tests (bimodality coefficient, Hartigan's dip test) find no bimodal structure — then scEMAIL defines the known group as all target cells and skips thresholding
- Raw source data are available and can be aligned with target data — the source-free, threshold-free design is not needed
- A priori known, manually specified thresholds are acceptable and user supervision is available

## Inputs

- Target-cell latent representations from the scEMAIL ZINB-based autoencoder source model
- Expert-ensemble E-scores (entropy EN, confidence CF, consistency CS) per target cell
- Confirmation that the E-score distribution is bimodal (bimodality coefficient and/or Hartigan's dip test)

## Outputs

- Adaptive threshold value delta (average E-score over all mixed cell pairs)
- Per-cell group assignments to 'known' (D) or 'unknown' (D_t), e.g. written to threshold_partition.csv

## How to apply

Load the target-cell latent representations and the expert-ensemble E-scores (combining entropy EN, confidence CF, and consistency CS) produced by the pre-trained source model, after bimodality testing has confirmed novel cell types. Construct synthetic manifold representations by mixing every distinct target-cell pair z_i, z_j with coefficient lambda ~ Uniform(0,1), following the manifold-mixup scheme. Compute the E-score of each synthetic mixed representation and set the adaptive threshold delta to the average E-score over all possible cell pairs. Partition cells with the rule: cell i is 'known' (group D) if E-score(z_i) > delta, otherwise 'unknown' (group D_t). This data-based partition avoids error propagation and overconfident predictions during subsequent network training with multi-order local affinity constraints.

## Related tools

- **scEMAIL** (Framework providing the expert ensemble E-score, manifold-mixup threshold computation, and known/unknown partition of target cells) — https://github.com/aster-ww/scEMAIL

## Examples

```
df['group'] = ['known' if e > delta else 'unknown' for e in df['e_score']]; df.to_csv('threshold_partition.csv', index=False)
```

## Evaluation signals

- Number of 'known' cells plus number of 'unknown' cells equals Nt (total target cells)
- Every cell with E-score > delta is labeled 'known' — exact match to the paper's partition rule
- Threshold delta equals the average E-score over all possible mixed pairs of target cells (lambda ~ Uniform(0,1) manifold mixup)
- Known/unknown grouping precedes and is consistent with bimodality-test confirmation (bimodality coefficient or Hartigan's dip test positive)
- Downstream adaptation uses local affinity regularizations only on 'known' cells, not on the unknown group

## Limitations

- If neither bimodality test indicates a bimodal E-score distribution, no thresholding is performed and all cells are treated as known — novel cell types may be missed
- Threshold quality depends on the manifold-mixup average over all cell pairs, which may be costly for large Nt
- The discussion of the article does not state explicit method limitations, and implementation details are deferred to File S1

## Evidence

- [intro] Partition target cells into known and unknown groups using an adaptive data-based threshold via manifold mixup when novel cell types are confirmed.: [evidence span withheld — anchor status: not_found]
- [methods] Adaptive threshold defined as average E-score over all possible target-cell pairs.: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [methods] Expert ensemble system with three components measures per-cell uncertainty.: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [readme] README states the adaptive manifold-mixup threshold replaces subjective user-selected thresholds.: "once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
