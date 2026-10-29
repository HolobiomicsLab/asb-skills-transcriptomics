---
name: known-unknown-cell-partitioning
description: Use when bimodality testing (bimodality coefficient and Hartigan's dip
  test) on target-cell E-scores confirms novel cell types, and cells must be split
  into known vs. unknown groups without a manually defined empirical threshold.
when_to_use_negative:
- Bimodality tests (bimodality coefficient and Hartigan's dip test) indicate no bimodal
  E-score structure — in that case all target cells are treated as known and thresholding
  is skipped
- Raw source data are available and explicit source-target feature alignment is intended
  (scEMAIL is a source-free adaptation method)
- Input is raw scRNA-seq counts rather than latent representations scored by a pre-trained
  source model with auxiliary classifiers
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scEMAIL
  role: 'Implements the novel cell-type perception module: E-score computation, bimodality
    testing, and the adaptive manifold-mixup threshold that partitions target cells
    into known and unknown groups'
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:72c87bcdb61b94f2ab05c13baa0df80322b97467dc3460148123a7131a1fcf06
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
n_claims: 10
---

# known-unknown-cell-partitioning

## Summary

Partitions target scRNA-seq cells into 'known' and 'unknown' groups using an adaptive, data-based threshold derived from manifold mixup of latent representations and expert-ensemble E-scores, avoiding user-selected thresholds when novel cell types are present.

## When to use

Use when bimodality testing (bimodality coefficient and Hartigan's dip test) on target-cell E-scores confirms novel cell types, and cells must be split into known vs. unknown groups without a manually defined empirical threshold.

## When NOT to use

- Bimodality tests (bimodality coefficient and Hartigan's dip test) indicate no bimodal E-score structure — in that case all target cells are treated as known and thresholding is skipped
- Raw source data are available and explicit source-target feature alignment is intended (scEMAIL is a source-free adaptation method)
- Input is raw scRNA-seq counts rather than latent representations scored by a pre-trained source model with auxiliary classifiers

## Inputs

- Target-cell latent representations from the ZINB-based autoencoder of the pre-trained source model
- Per-cell expert-ensemble E-scores (entropy, confidence, consistency) from the source model and its two auxiliary co-training classifiers
- Bimodality test results (bimodality coefficient and Hartigan's dip test) on the E-score distribution

## Outputs

- Per-cell known/unknown group assignments
- Adaptive threshold value delta (average E-score over all mixed cell pairs)
- threshold_partition.csv containing per-cell group assignments together with the threshold value

## How to apply

First compute the expert-ensemble E-score (entropy, confidence, consistency) for each target-cell latent representation under the pre-trained source model, and run the bimodality coefficient and Hartigan's dip test; if neither indicates bimodality, assign all target cells to the known group and skip thresholding. If bimodality is confirmed, construct synthetic manifold representations by mixing every pair of distinct target cells z_i and z_j with coefficient lambda drawn from Uniform(0,1) per the manifold-mixup scheme. Compute the E-score of each synthetic mixed representation and set the threshold delta to the average E-score over all possible cell pairs. Partition with the rule: cell i is in the known group D if E-score(z_i) > delta, otherwise in the unknown group D_t. The rationale is that mixed synthetic cells should be no more uncertain than the boundary between populations, so the average mixed-pair E-score is a data-driven decision boundary; this partition also prevents error propagation and overconfident predictions in subsequent local-affinity network training.

## Related tools

- **scEMAIL** (Implements the novel cell-type perception module: E-score computation, bimodality testing, and the adaptive manifold-mixup threshold that partitions target cells into known and unknown groups) — https://github.com/aster-ww/scEMAIL

## Examples

```
python scripts/threshold_partition.py --escores target_escores.csv --latents target_latents.npy --mixup uniform --output threshold_partition.csv
```

## Evaluation signals

- Number of known cells plus number of unknown cells equals Nt (the total target-cell count)
- Every cell with E-score > delta is labeled known, exactly matching the paper's partition rule E-score(z_i) > delta
- The threshold delta equals the average E-score over all pairs of mixed synthetic representations (lambda ~ Uniform(0,1))
- Thresholding is only performed when the bimodality coefficient or Hartigan's dip test indicates a bimodal E-score distribution; otherwise all cells are known
- Downstream model adaptation shows no overconfident predictions or error propagation from unlabeled novel cell types being treated as known

## Limitations

- The threshold is only applied after bimodality is confirmed; if neither the bimodality coefficient nor Hartigan's dip test indicates bimodality, no novel cell types are assumed and thresholding is skipped
- Method assumes a pre-trained source model is available with two auxiliary co-training classifiers; consistency relies on it being unlikely that both classifiers wrongly predict a cell into the same type
- Implementation details are deferred to File S1 in the supplement rather than fully specified in the main text

## Evidence

- [readme] an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "Third, once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
- [intro] target cells are then separated into "known" and "unknown" groups based on an adaptive data-based threshold by manifold mixup.: [evidence span withheld — anchor status: not_found]
- [methods] The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
