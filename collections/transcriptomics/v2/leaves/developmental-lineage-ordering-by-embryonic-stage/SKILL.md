---
name: developmental-lineage-ordering-by-embryonic-stage
description: Use when SCASL clusters of a developmental scRNA-seq dataset (e.g. 486-cell
  embryonic hepatoblasts) need biological validation by checking whether cluster orderings
  recapitulate known lineage progression across embryonic days.
when_to_use_negative:
- The goal is cell-type classification from gene expression alone — SCASL clusters
  on alternative splicing landscapes, and most findings are not available via conventional
  gene-expression-based clustering
- The dataset lacks developmental time points or known lineage structure, so cluster
  orderings cannot be validated against embryonic-day progression
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: Clusters cells on alternative-splicing landscapes to produce AS cluster labels
    whose ordering is compared against embryonic-day lineage progression
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Identifies junction reads to define AS events for the SCASL AS probability
    matrix underlying the clusters
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:f2df9d6e64c22c33dd54119610e97b6b9561624e07d71d3ca74c1a3c637bb962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# developmental-lineage-ordering-by-embryonic-stage

## Summary

Validates SCASL-derived alternative-splicing clusters by ordering them along embryonic hepatoblast differentiation lineages, confirming recovered sequences such as C2-C10-C3-C1-C8-C5 (hepatocyte) and C0-C9-C6 (cholangiocyte) against embryonic-day progression.

## When to use

Use when SCASL clusters of a developmental scRNA-seq dataset (e.g. 486-cell embryonic hepatoblasts) need biological validation by checking whether cluster orderings recapitulate known lineage progression across embryonic days.

## When NOT to use

- The goal is cell-type classification from gene expression alone — SCASL clusters on alternative splicing landscapes, and most findings are not available via conventional gene-expression-based clustering
- The dataset lacks developmental time points or known lineage structure, so cluster orderings cannot be validated against embryonic-day progression

## Inputs

- 486-cell embryonic hepatoblast scRNA-seq dataset (bam or junction input files)
- YAML configuration file (configs/srr.yaml) with QC parameters, pca_dimension, num_cluster, and dimension_reduction_method

## Outputs

- SCASL cluster label file in which 'preds' gives the AS cluster label per cell
- reduce.png (UMAP/t-SNE colored by embryonic-day ground truth) and cluster.png (colored by SCASL clusters)
- Validated lineage orderings: hepatocyte C2-C10-C3-C1-C8-C5 and cholangiocyte C0-C9-C6

## How to apply

Run SCASL on the embryonic hepatoblast dataset via 'python main.py -y configs/srr.yaml' with configured QC parameters (sites_initial, runs_initial, sites_quality, runs_quality), pca_dimension, and num_cluster to obtain AS cluster labels (preds). Visualize the clusters with the configured dimension_reduction_method ('umap' or 'tsne'), producing reduce.png colored by embryonic day (ground truth) and cluster.png colored by SCASL clusters. Compare the SCASL cluster assignments against the reported hepatocyte sequence C2-C10-C3-C1-C8-C5 and cholangiocyte sequence C0-C9-C6 (Figs. 4A, S6A), checking that consecutive clusters align with embryonic-day progression. Interpret the recovered cluster ordering as recapitulation of hepatoblast-to-hepatocyte and hepatoblast-to-cholangiocyte differentiation trajectories. Accept minor run-to-run differences arising from imputation and unsupervised clustering randomness, provided the overall lineage ordering remains consistent for identical parameters.

## Related tools

- **SCASL** (Clusters cells on alternative-splicing landscapes to produce AS cluster labels whose ordering is compared against embryonic-day lineage progression) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Identifies junction reads to define AS events for the SCASL AS probability matrix underlying the clusters) — https://github.com/davidaknowles/leafcutter

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- Recovered clusters recapitulate the reported hepatocyte sequence C2-C10-C3-C1-C8-C5 and cholangiocyte sequence C0-C9-C6 (Figs. 4A, S6A)
- Cluster ordering is consistent with embryonic-day progression shown in reduce.png colored by ground truth
- Output cluster label file exists and 'preds' assigns an AS cluster label to each of the 486 cells
- Results remain generally consistent across repeated runs with identical parameters, allowing only minor randomness-driven differences
- lineage clusters correspond to biologically expected hepatocyte vs. cholangiocyte branches from hepatoblasts

## Limitations

- Imputation, dimensionality reduction, and unsupervised clustering are stochastic, so run-to-run differences may occur even with identical parameters
- RNA splicing profiling is much more costly than simple gene-expression quantification under current single-cell transcriptome profiling practice
- SMART-seq-style input data is costly and difficult to multiplex for large cell numbers

## Evidence

- [results] hepatocyte ordering: [evidence span withheld — anchor status: not_found]
- [abstract] lineage finding from abstract: "illustrates cell lineages of embryonic liver development"
- [discussion] imputation/clustering method basis: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [readme] run invocation: "> python main.py -y configs/srr.yaml"
- [readme] randomness caveat: "Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results"
