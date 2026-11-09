---
name: cross-method-annotation-benchmark-comparison
description: Use when comparing source-free scRNA-seq annotation methods on target
  datasets containing target-private (novel) cell types, to test whether adaptive
  unknown-cell detection outperforms user-defined thresholds.
when_to_use_negative:
- Target data contain no novel cell types (closed/partial settings without target-private
  types) — the bimodality tests and known/unknown partitioning are not triggered and
  threshold comparison is not informative
- Raw source data are available and explicit source-target feature alignment is possible
  — a non-source-free benchmark framing is more appropriate
- You need only overall annotation accuracy (e.g., accuracy/H-score) without evaluating
  novel-cell detection outcomes
tools:
- name: scEMAIL
  role: 'Primary method under benchmark: its expert-ensemble E-score, bimodality tests,
    and adaptive manifold-mixup threshold produce the known/unknown partition being
    evaluated'
  repo: https://github.com/aster-ww/scEMAIL
- name: scmap
  role: 'Competitor baseline: assigns cell types by maximum similarity to known cluster
    centroids or cells in the source, relying on manual thresholds for novel cells'
- name: ItClust
  role: 'Competitor baseline: iterative deep transfer learning model initialized from
    well-annotated source data'
- name: scSemiCluster
  role: 'Competitor baseline: adds structural similarity regularization to source
    data to constrain target-data clustering results'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:24855b6b1d4764da9188abe81b392c40a81a5fbdeaa5ddd119cca1a368769407
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-method-annotation-benchmark-comparison

## Summary

Benchmarks scEMAIL's novel cell-type perception module (E-score, bimodality tests, adaptive manifold-mixup threshold) against manually thresholded annotation competitors (scmap, ItClust, scSemiCluster) on open and open-partial scRNA-seq settings, quantifying unknown-cell detection rather than only classification accuracy.

## When to use

Use when comparing source-free scRNA-seq annotation methods on target datasets containing target-private (novel) cell types, to test whether adaptive unknown-cell detection outperforms user-defined thresholds.

## When NOT to use

- Target data contain no novel cell types (closed/partial settings without target-private types) — the bimodality tests and known/unknown partitioning are not triggered and threshold comparison is not informative
- Raw source data are available and explicit source-target feature alignment is possible — a non-source-free benchmark framing is more appropriate
- You need only overall annotation accuracy (e.g., accuracy/H-score) without evaluating novel-cell detection outcomes

## Inputs

- Per-cell E-score distributions for the target dataset(s) (e.g., open-setting 'mammary gland', open-partial-setting 'neonatal rib')
- Latent representations z_i of target cells from the scEMAIL autoencoder (for manifold-mixup thresholding)
- Reported open/open-partial benchmark annotation outputs for scmap, ItClust, and scSemiCluster
- Ground-truth labels marking target-private (novel) cell types for precision/recall of unknown detection

## Outputs

- Per-dataset, per-method unknown-cell detection results (precision/recall of the unknown set vs. true target-private cell types)
- novel_cell_type_detection_comparison.csv containing all four methods on both the open and open-partial datasets
- Per-cell known/unknown assignment table (known set D with E-score > delta; unknown set D_t with E-score <= delta)

## How to apply

Load precomputed per-cell E-score distributions for the open-setting 'mammary gland' and open-partial-setting 'neonatal rib' target datasets; the E-score is an expert ensemble of three components: entropy (EN, normalized entropy divided by log K), confidence (CF), and consistency (CS) across two auxiliary co-training classifiers, averaged over three experts. Run two bimodality measures — the bimodality coefficient and Hartigan's dip test — on the empirical E-score distribution and declare novel cell types present if at least one measure indicates bimodality. For flagged datasets, build synthetic manifold points by mixing every pair of latent representations z_i and z_j with lambda drawn from Uniform(0,1), and set the adaptive threshold delta to the average E-score over all pairs, partitioning cells into known (E-score > delta) and unknown (E-score <= delta) sets. Compute unknown-cell detection precision/recall of the unknown set against true target-private cell types for scEMAIL, and assemble comparable detection outcomes for scmap, ItClust, and scSemiCluster from the reported open/open-partial benchmark outputs. The rationale is that partitioning into known and unknown groups during model adaptation increases classification accuracy when novel cell types exist, avoiding subjective threshold selection.

## Related tools

- **scEMAIL** (Primary method under benchmark: its expert-ensemble E-score, bimodality tests, and adaptive manifold-mixup threshold produce the known/unknown partition being evaluated) — https://github.com/aster-ww/scEMAIL
- **scmap** (Competitor baseline: assigns cell types by maximum similarity to known cluster centroids or cells in the source, relying on manual thresholds for novel cells)
- **ItClust** (Competitor baseline: iterative deep transfer learning model initialized from well-annotated source data)
- **scSemiCluster** (Competitor baseline: adds structural similarity regularization to source data to constrain target-data clustering results)

## Evaluation signals

- novel_cell_type_detection_comparison.csv contains rows for all four methods (scEMAIL, scmap, ItClust, scSemiCluster) on both the open ('mammary gland') and open-partial ('neonatal rib') datasets
- Bimodality is declared only when at least one of the bimodality coefficient or Hartigan's dip test indicates a bimodal E-score distribution
- Adaptive threshold delta equals the average E-score over all mixture pairs with lambda drawn from Uniform(0,1), not a hand-picked constant
- Results reproduce the paper's pattern: scEMAIL maintains stable novel-type detection across settings while manually thresholded competitors degrade when target-private cell types exist
- Known/unknown partition is monotone in the E-score: cells with E-score > delta enter the known set D, cells with E-score <= delta enter the unknown set D_t

## Limitations

- Implementation details of competing methods are only in supplementary Table S1, so benchmark comparability depends on those external settings
- Bimodality declaration relies on empirical E-score distributions showing apparent bimodal structure; the method presumes this structure emerges when novel cell types exist
- Competing approaches depend on subjective, manually selected thresholds, which can bias detection outcomes in either direction depending on threshold choice

## Evidence

- [intro] we propose scEMAIL, a universal transfer learning-based annotation framework for scRNA-seq data, which incorporates expert ensemble novel cell-type perception and local affinity constraints: "we propose scEMAIL, a universal transfer learning-based annotation framework for scRNA-seq data, which incorporates expert ensemble novel cell-type perception and local affinity constraints"
- [methods] we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS).: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process.: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [abstract] an adaptive threshold via manifold mixup partitions target cells into "known" and "unknown" groups: "an adaptive threshold via manifold mixup partitions target cells into "known" and "unknown" groups"
- [readme] A subjective threshold is often selected by users to detect novel cells.: "A subjective threshold is often selected by users to detect novel cells."
