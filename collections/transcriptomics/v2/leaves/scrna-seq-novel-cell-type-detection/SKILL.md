---
name: scrna-seq-novel-cell-type-detection
description: Use when annotating target scRNA-seq data under open or open-partial
  settings where target-private cell types may exist, raw source data are unavailable,
  and no prior threshold for 'unknown' detection is known.
when_to_use_negative:
- Target data are known to contain no novel cell types (closed or partial label-overlap
  setting) — bimodality testing and threshold partitioning are unnecessary
- Raw labeled source data are available and the user prefers explicit source-target
  feature alignment methods (e.g. scArches) rather than source-free adaptation
- Only per-cluster annotations without cell-level uncertainty are needed — the E-score
  requires per-cell classifier probability outputs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: 'Implements the full novel cell-type perception module: expert ensemble E-score
    computation, bimodality testing, and adaptive manifold-mixup thresholding'
  repo: https://github.com/aster-ww/scEMAIL
- name: scmap
  role: Manual-threshold competitor benchmark for unknown-cell detection rates comparison
- name: ItClust
  role: Manual-threshold competitor benchmark for unknown-cell detection rates comparison
- name: scSemiCluster
  role: Manual-threshold competitor benchmark for unknown-cell detection rates comparison
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:99a3788164f379a3338195582c6427d2742f2aae390a445665c8233f338f1f0c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrna-seq-novel-cell-type-detection

## Summary

Detects novel cell types in target scRNA-seq data without source data or manual thresholds by testing the expert-ensemble E-score distribution for bimodality (bimodality coefficient + Hartigan's dip test) and, if bimodal, partitioning cells into 'known' and 'unknown' groups via an adaptive manifold-mixup threshold.

## When to use

Use when annotating target scRNA-seq data under open or open-partial settings where target-private cell types may exist, raw source data are unavailable, and no prior threshold for 'unknown' detection is known.

## When NOT to use

- Target data are known to contain no novel cell types (closed or partial label-overlap setting) — bimodality testing and threshold partitioning are unnecessary
- Raw labeled source data are available and the user prefers explicit source-target feature alignment methods (e.g. scArches) rather than source-free adaptation
- Only per-cluster annotations without cell-level uncertainty are needed — the E-score requires per-cell classifier probability outputs

## Inputs

- Pre-computed per-cell E-score values from the scEMAIL expert ensemble (e.g. e_scores.csv) for the target scRNA-seq dataset
- Latent representations z_i of target cells from the scEMAIL ZINB autoencoder encoder
- Soft probability outputs of the two auxiliary co-training classifiers (for consistency)

## Outputs

- bimodality_decision file (JSON) containing the binary novel-cell-type flag plus the bimodality coefficient and Hartigan's dip test statistics
- Known/unknown cell partition: known set D (E-score > δ) and unknown set Dt (E-score <= δ) with per-cell assignments
- novel_cell_type_detection_comparison.csv with per-dataset, per-method unknown-cell detection precision/recall

## How to apply

First compute the per-cell expert ensemble E-score (average of normalized entropy EN/logK, maximum confidence, and consistency — the dot product of the two auxiliary co-training classifiers' soft probability vectors; Eq. 8) for all target cells. Then test the empirical E-score distribution for bimodality using both the bimodality coefficient (formulation in File S1) and Hartigan's dip test; if either indicates a bimodal structure, declare novel cell types present (flag = 1). When bimodality is confirmed, construct synthetic manifold-mixup points z~ = λ*z_i + (1-λ)*z_j with λ ~ Uniform(0,1) and set the adaptive threshold δ to the average E-score over all target-cell pairs (Eq. 10). Partition cells with E-score > δ into the known set D and E-score <= δ into the unknown set Dt, labeling Dt cells 'unknown' during downstream annotation; update the partition each iteration during model adaptation. This adaptive, data-based partition outperforms manually defined thresholds used by competitors like scmap, ItClust, and scSemiCluster.

## Related tools

- **scEMAIL** (Implements the full novel cell-type perception module: expert ensemble E-score computation, bimodality testing, and adaptive manifold-mixup thresholding) — https://github.com/aster-ww/scEMAIL
- **scmap** (Manual-threshold competitor benchmark for unknown-cell detection rates comparison)
- **ItClust** (Manual-threshold competitor benchmark for unknown-cell detection rates comparison)
- **scSemiCluster** (Manual-threshold competitor benchmark for unknown-cell detection rates comparison)

## Examples

```
python bimodality_test.py --e_scores e_scores.csv --out bimodality_decision.
```

## Evaluation signals

- bimodality_decision file exists, is valid JSON, and contains both the binary decision field and the two test statistics
- Decision flag is consistent with the either-test-positive rule: bimodality coefficient OR dip test indicating bimodality yields flag = 1
- When novel cell types exist, partitioning into D and Dt increases classification accuracy relative to subjective thresholding (ablation expectation)
- Detection comparison table covers all four methods on both the open ('mammary gland') and open-partial ('neonatal rib') datasets
- scEMAIL maintains stable novel-type detection across settings while manually thresholded competitors (scmap, ItClust, scSemiCluster) degrade when target-private cell types exist

## Limitations

- The threshold δ is data-derived via manifold mixup, so if the E-score distribution is unimodal yet novel types exist subtly, the module may miss them (depends on bimodality tests' power)
- Implementation details of bimodality coefficient and dip test are deferred to File S1, not fully specified in the main text
- Requires the pre-trained source model with two auxiliary co-training classifiers; cannot run on a single-classifier pipeline

## Evidence

- [methods] E-score bimodality with novel cell types: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [methods] Either-test-positive rule: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] Adaptive manifold-mixup threshold: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [results] Ablation vs manual thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Three-step perception module (README): "Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types."
