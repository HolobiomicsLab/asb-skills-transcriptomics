---
name: bimodality-test-interpretation
description: Use when annotating a target scRNA-seq dataset in open or open-partial
  settings and you must decide, from the E-score (entropy/confidence/consistency)
  distribution alone, whether target-private cell types exist before partitioning
  cells into known vs unknown groups.
when_to_use_negative:
- The setting is closed or partial with no possibility of target-private cell types,
  so threshold-free novel-type detection is unnecessary
- The target data contain only one cell type or the E-score distribution is unimodal
  by design, making bimodality tests uninformative
- Raw source data are available and you intend to perform explicit source-target feature
  alignment (e.g. scmap, ItClust-style transfer) rather than source-free adaptation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Provides the E-score expert ensemble whose distribution the bimodality tests
    are run on, and implements the manifold-mixup adaptive threshold and known/unknown
    partitioning
  repo: https://github.com/aster-ww/scEMAIL
- name: scmap
  role: Manual-threshold competitor baseline whose novel-cell detection degrades in
    open settings, used for comparison
- name: ItClust
  role: Manual-threshold competitor baseline for novel-cell detection comparison in
    open/open-partial settings
- name: scSemiCluster
  role: Manual-threshold competitor baseline for novel-cell detection comparison in
    open/open-partial settings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:9bfedc3100bef3d35bc344ccc797f1b20adbe1389fa2e83dd9562723bb4b9e2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - ItClust
  - scEMAIL
  - scSemiCluster
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bimodality-test-interpretation

## Summary

Applies the bimodality coefficient and Hartigan's dip test to the empirical distribution of scEMAIL's expert-ensemble uncertainty scores (E-scores) to decide whether target scRNA-seq data contain novel cell types, replacing user-defined thresholds with a data-driven decision.

## When to use

Use when annotating a target scRNA-seq dataset in open or open-partial settings and you must decide, from the E-score (entropy/confidence/consistency) distribution alone, whether target-private cell types exist before partitioning cells into known vs unknown groups.

## When NOT to use

- The setting is closed or partial with no possibility of target-private cell types, so threshold-free novel-type detection is unnecessary
- The target data contain only one cell type or the E-score distribution is unimodal by design, making bimodality tests uninformative
- Raw source data are available and you intend to perform explicit source-target feature alignment (e.g. scmap, ItClust-style transfer) rather than source-free adaptation

## Inputs

- Per-cell E-score vector for the target dataset (expert ensemble of normalized entropy / log K, max-confidence, and auxiliary-classifier consistency averaged over three experts)
- Latent representations z_i of target cells (for the manifold-mixup adaptive threshold)
- Target scRNA-seq datasets from open ('mammary gland') or open-partial ('neonatal rib') settings

## Outputs

- Binary decision: novel cell types present vs absent (from bimodality coefficient / Hartigan's dip test)
- Adaptive threshold delta (mean E-score over all manifold-mixup pairs)
- Per-cell partition labels: known set D (E-score > delta) and unknown set D_t (E-score <= delta)
- Novel-cell-type detection precision/recall table (e.g. novel_cell_type_detection_comparison.csv)

## How to apply

First compute the per-cell E-score for the target data as the average over three experts of normalized entropy divided by log K, max-confidence, and auxiliary-classifier consistency. Then run two bimodality measures — the bimodality coefficient and Hartigan's dip test — on the empirical E-score distribution; if at least one measure indicates bimodality, declare that novel cell types are present. Once bimodality is confirmed, construct synthetic manifold points by mixing every pair of latent representations z_i and z_j with lambda drawn from Uniform(0,1), and set the adaptive threshold delta to the average E-score over all pairs. Finally, partition target cells into the known set D (E-score > delta) and the unknown set D_t (E-score <= delta) for downstream annotation as 'unknown'. The rationale is that target data containing novel cell types exhibit an apparent bimodal E-score structure, so the presence of a second mode is itself the detection signal rather than a subjective cutoff.

## Related tools

- **scEMAIL** (Provides the E-score expert ensemble whose distribution the bimodality tests are run on, and implements the manifold-mixup adaptive threshold and known/unknown partitioning) — https://github.com/aster-ww/scEMAIL
- **scmap** (Manual-threshold competitor baseline whose novel-cell detection degrades in open settings, used for comparison)
- **ItClust** (Manual-threshold competitor baseline for novel-cell detection comparison in open/open-partial settings)
- **scSemiCluster** (Manual-threshold competitor baseline for novel-cell detection comparison in open/open-partial settings)

## Examples

```
from diptest import diptest; dip_stat, dip_p = diptest(np.asarray(e_scores)); novel_present = dip_p < 0.05 or bimodality_coefficient(e_scores) exceeds threshold
```

## Evaluation signals

- Both the bimodality coefficient and Hartigan's dip test are computed, and a novel-type-positive decision requires only one measure to indicate bimodality
- The adaptive threshold delta equals the mean E-score over all mixup pairs (lambda ~ Uniform(0,1)), not a hand-set value
- The output table contains all four methods (scEMAIL, scmap, ItClust, scSemiCluster) on both the open ('mammary gland') and open-partial ('neonatal rib') datasets
- The reported pattern is reproduced: scEMAIL maintains stable novel-type detection across settings while manually thresholded competitors degrade when target-private cell types exist
- Ablation shows that partitioning into known/unknown groups during adaptation increases classification accuracy relative to no partitioning

## Limitations

- If neither bimodality measure flags the distribution, novel cell types are assumed absent, so a genuinely novel but low-separability type may be missed
- Implementation details of competing methods are deferred to supplementary Table S1, and simulated-experiment details to File S1, limiting reproducibility from the main text alone
- Detection outcomes for competitors depend on their manually defined thresholds, so cross-method comparisons inherit those subjective choices

## Evidence

- [methods] When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [readme] an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups.: "an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
