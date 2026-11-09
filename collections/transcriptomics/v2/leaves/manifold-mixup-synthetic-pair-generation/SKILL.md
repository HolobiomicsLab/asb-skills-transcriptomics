---
name: manifold-mixup-synthetic-pair-generation
description: Use when bimodality testing (bimodality coefficient or Hartigan's dip
  test) has confirmed novel cell types in target scRNA-seq data and cells must be
  split into known vs. unknown groups without a manually chosen E-score threshold.
when_to_use_negative:
- Bimodality tests (bimodality coefficient, Hartigan's dip test) do not indicate a
  bimodal E-score distribution — then all target cells are defined as the known group
  and thresholding is skipped
- Source data are available and a supervised annotation pipeline with an explicitly
  user-selected novel-cell threshold is acceptable
- Input is raw count matrices rather than source-model latent representations with
  computed E-scores — mixing must operate on the manifold embeddings
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Framework implementing the expert ensemble E-scores, bimodality tests, and
    manifold-mixup adaptive threshold that partitions target cells into known/unknown
    groups
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:fa0f9f6cfd7b933e14a4bbeb889b712c029d8dba787ce272395b7852b0a3d4e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# manifold-mixup-synthetic-pair-generation

## Summary

An adaptive, data-based thresholding procedure from scEMAIL that generates synthetic representations by mixing every pair of target-cell latent embeddings via manifold mixup, averages their E-scores to set the threshold delta, and partitions target cells into 'known' and 'unknown' groups without a user-selected cutoff.

## When to use

Use when bimodality testing (bimodality coefficient or Hartigan's dip test) has confirmed novel cell types in target scRNA-seq data and cells must be split into known vs. unknown groups without a manually chosen E-score threshold.

## When NOT to use

- Bimodality tests (bimodality coefficient, Hartigan's dip test) do not indicate a bimodal E-score distribution — then all target cells are defined as the known group and thresholding is skipped
- Source data are available and a supervised annotation pipeline with an explicitly user-selected novel-cell threshold is acceptable
- Input is raw count matrices rather than source-model latent representations with computed E-scores — mixing must operate on the manifold embeddings

## Inputs

- Target-cell latent representations z_i from the scEMAIL source model (ZINB-based autoencoder features)
- Per-cell expert-ensemble E-scores combining entropy (EN), confidence (CF), and consistency (CS)
- Bimodality test results (bimodality coefficient and Hartigan's dip test) confirming novel cell types

## Outputs

- Adaptive threshold value delta (average E-score over all mixed cell pairs)
- Per-cell group assignments: known group D (E-score > delta) vs. unknown group D_t (E-score <= delta)
- Per-cell group assignments table written to threshold_partition.csv together with the threshold value

## How to apply

First load the target-cell latent representations and expert-ensemble E-scores (combining entropy, confidence, and consistency) computed by the source model, and confirm the E-score distribution is bimodal; if neither the bimodality coefficient nor Hartigan's dip test signals bimodality, treat all target cells as known and skip thresholding. Then construct synthetic manifold representations by mixing every pair of distinct target cells z_i and z_j with a coefficient lambda drawn from Uniform(0,1), per the manifold-mixup scheme. Compute the E-score of each synthetic mixed representation and set the threshold delta to the average E-score over all possible cell pairs, so the cutoff is derived from the data itself rather than fixed empirically. Finally, partition the target data by the rule: cell i belongs to the known group D if E-score(z_i) > delta, otherwise to the unknown group D_t; this partition prevents error propagation and overconfident predictions during subsequent network training with local affinity constraints.

## Related tools

- **scEMAIL** (Framework implementing the expert ensemble E-scores, bimodality tests, and manifold-mixup adaptive threshold that partitions target cells into known/unknown groups) — https://github.com/aster-ww/scEMAIL

## Examples

```
lambda = np.random.uniform(0, 1); z_mix = lambda * z_i + (1 - lambda) * z_j; delta = np.mean([e_score(l * a + (1 - l) * b) for a, b in pairs(a != b) for l in np.random.uniform(0, 1)])
```

## Evaluation signals

- Number of known cells plus number of unknown cells equals Nt (the total number of target cells)
- Every cell with E-score > delta is labeled known, exactly matching the paper's partition rule
- Delta equals the average E-score over all synthetic pairs, not a hand-picked empirical value
- E-score distribution of target cells exhibits the expected bimodal structure before thresholding (verified by bimodality coefficient or Hartigan's dip test)
- Synthetic pairs are generated only from distinct target-cell pairs with lambda ~ Uniform(0,1)

## Limitations

- If neither the bimodality coefficient nor Hartigan's dip test indicates bimodality, the threshold step is skipped and all cells are assumed known — the skill cannot confirm novel types on its own
- The thresholding depends on quality of the upstream E-score; consistency is robust to batch effect but entropy and confidence may be affected by source-model prediction errors
- Implementation details beyond Algorithm 1 are deferred to File S1, so exact parameter handling may require consulting the supplementary material or code

## Evidence

- [intro] target cells are then separated into "known" and "unknown" groups based on an adaptive data-based threshold by manifold mixup.: [evidence span withheld — anchor status: not_found]
- [methods] The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [readme] Third, once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "Third, once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
