---
name: cross-method-accuracy-benchmarking
description: Use when evaluating a scRNA-seq cell-type annotation method head-to-head
  with published baselines under multiple label-overlap settings, and accuracy values
  must be matched against a reference table.
when_to_use_negative:
- When raw source data are available and explicit source-target feature alignment
  is intended — scEMAIL is designed for the source-free regime where raw source data
  are unavailable
- When the goal is annotating data types outside the validated scope (e.g., non-single-cell
  bulk data); the paper only demonstrates scRNA-seq, CITE-seq, and scATAC-seq
- When novel cell types are known a priori and a manual threshold suffices — the skill's
  adaptive E-score/bimodality pipeline is meant to replace manually defined thresholds
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: 'Method under test: source-free annotation with novel cell-type perception
    via E-score, bimodality tests, and multi-order local affinity constraints'
  repo: https://github.com/aster-ww/scEMAIL
- name: scmap
  role: 'Baseline competitor: assigns cell types by maximum similarity between source
    cluster centroids or cells and target samples'
- name: Seurat v3
  role: 'Baseline competitor: constructs a shared nearest neighbor graph from pairwise
    cell relationships to identify cell types'
- name: ItClust
  role: 'Baseline competitor: iterative deep transfer learning initialized from well-annotated
    source data'
- name: scSemiCluster
  role: 'Baseline competitor: adds structural similarity regularization on source
    data to constrain target clustering'
- name: scArches
  role: 'Baseline competitor: transfer learning via single-cell architectural surgery
    mapping target onto source without sharing raw source data'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:1e51f6dc52ed4fb82c53d706adca9ee9ce7b957d1cb59a6e1ddde8c403a5720f
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cross-method-accuracy-benchmarking

## Summary

Reproduce and compare cell-type annotation accuracy of scEMAIL against competing methods (scmap, Seurat v3, ItClust, scSemiCluster, scArches, source-only) across the four label-overlap settings (closed, partial, open, open-partial) on real scRNA-seq dataset pairs, using annotation accuracy as the metric.

## When to use

Use when evaluating a scRNA-seq cell-type annotation method head-to-head with published baselines under multiple label-overlap settings, and accuracy values must be matched against a reference table.

## When NOT to use

- When raw source data are available and explicit source-target feature alignment is intended — scEMAIL is designed for the source-free regime where raw source data are unavailable
- When the goal is annotating data types outside the validated scope (e.g., non-single-cell bulk data); the paper only demonstrates scRNA-seq, CITE-seq, and scATAC-seq
- When novel cell types are known a priori and a manual threshold suffices — the skill's adaptive E-score/bimodality pipeline is meant to replace manually defined thresholds

## Inputs

- Labeled source scRNA-seq dataset and target scRNA-seq dataset pair (e.g., bone marrow under closed setting, pancreas under partial setting, mammary gland under open setting, neonatal rib under open-partial setting)
- Pre-trained scEMAIL source model weights (ZINB autoencoder + classifier + two auxiliary classifiers)
- Repository code and supplementary implementation details (File S1, Table S1) for baselines

## Outputs

- Annotation accuracy per method per dataset pair and setting, compiled as a comparison table (Table 1 style)
- Per-cell label assignments, with cells partitioned into known (argmax of classifier c) and 'unknown' (Dt group)

## How to apply

Pre-train the scEMAIL source model with a ZINB model-based autoencoder (negative log-likelihood reconstruction loss), a supervised cross-entropy classifier, and two auxiliary co-training classifiers; then run the expert ensemble E-score (normalized entropy, maximum confidence, auxiliary-classifier consistency), bimodality tests (bimodality coefficient and Hartigan's dip test), manifold-mixup adaptive threshold, and encoder-only fine-tuning with multi-order local affinity loss (unidirectional alpha = 0.1). Assign known cells by classifier argmax and 'unknown' to the Dt group, then benchmark against scmap, Seurat v3, ItClust, scSemiCluster, scArches, and source-only using annotation accuracy on each dataset pair. Compile a Table-1-style comparison across the four settings; implementation details of competing methods are in supplementary Table S1. Validation requires scEMAIL to rank highest and the reproduced accuracies to match reported reference points (e.g., bone marrow 0.8873, trachea 0.9415) within tolerance.

## Related tools

- **scEMAIL** (Method under test: source-free annotation with novel cell-type perception via E-score, bimodality tests, and multi-order local affinity constraints) — https://github.com/aster-ww/scEMAIL
- **scmap** (Baseline competitor: assigns cell types by maximum similarity between source cluster centroids or cells and target samples)
- **Seurat v3** (Baseline competitor: constructs a shared nearest neighbor graph from pairwise cell relationships to identify cell types)
- **ItClust** (Baseline competitor: iterative deep transfer learning initialized from well-annotated source data)
- **scSemiCluster** (Baseline competitor: adds structural similarity regularization on source data to constrain target clustering)
- **scArches** (Baseline competitor: transfer learning via single-cell architectural surgery mapping target onto source without sharing raw source data)

## Examples

```
git clone https://github.com/aster-ww/scEMAIL && python run_scEMAIL.py --source bone_marrow_source --target bone_marrow_target --setting closed; python benchmark.py --baselines scmap,seurat3,itclust,scSemiCluster,scArches,source_only --metric accuracy
```

## Evaluation signals

- Reproduced scEMAIL total annotation accuracy matches reported values within tolerance (e.g., bone marrow 0.8873, trachea 0.9415 in Table 1)
- scEMAIL ranks highest among all competing methods across closed, partial, open, and open-partial settings
- scEMAIL total annotation accuracy exceeds 0.7 on the real scRNA-seq dataset pairs (bone marrow, pancreas, mammary gland, neonatal rib)
- Known/unknown partition is reproducible: adaptive threshold delta equals the mean E-score over target cell pairs from manifold mixup (Eq. 10)
- Bimodality decision agrees between the bimodality coefficient and Hartigan's dip test on the empirical E-score distribution

## Limitations

- Implementation details of competing methods and simulation experiments are deferred to supplementary Table S1 and File S1, not the main text
- Annotation accuracy for the scATAC-seq 'mouse bone marrow' experiment is only shown in Figure 10, not reported numerically in the text
- The paper's discussion states no explicit limitations of the method; edge-case behavior beyond the tested dataset pairs is undocumented

## Evidence

- [results] scEMAIL performed best among all competing methods, achieving total annotation accuracies greater than 0.7 on the real scRNA-seq dataset pairs: [evidence span withheld — anchor status: not_found]
- [results] Bone marrow table row with accuracies: "Bone marrow0.85510.50930.85640.84980.84360.83880.8873"
- [results] Implementation details deferred to supplementary: "The implementation details of competing methods can be seen in Table S1."
- [methods] Bimodality tests for novel cell types: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [readme] README reference to paper: "For more information, please refer to https://doi.org/10.1016/j.gpb.2022.12.008"
