---
name: replicate-cluster-matching-across-biological-datasets
description: Use when two or more independently clustered scRNAseq datasets (e.g.,
  replicates, tissues, or disease models like TIL vs dLN vs Arm CD4+ T cells) must
  be compared to find which clusters share transcriptomic profiles and which remain
  distinct.
when_to_use_negative:
- Datasets have not yet been individually clustered — the method requires precomputed
  per-dataset clusters and marker fold-change statistics as input
- The goal is de novo joint clustering on a single pooled matrix rather than matching
  independently derived clusters across pre-clustered datasets
- Only one dataset (no replicate or cross-condition comparison) is available, leaving
  no second set of clusters to correlate against
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: robustSingleCell
  role: Provides assess.cluster.similarity, get.robust.cluster.similarity, visualize.cluster.cors.heatmaps,
    and visualize.cluster.similarity.stats used to compute and filter cross-dataset
    cluster correlations
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell pipeline and its correlation
    analysis are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:6392c813000d5538c14c208fa8bdb16cc87f9ae9f0221a072df2e66ae9a9cd69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# replicate-cluster-matching-across-biological-datasets

## Summary

A cross-dataset cluster similarity analysis from robustSingleCell (Magen et al. 2019) that matches cell subpopulations identified independently in two or more scRNAseq datasets (e.g., tumor-infiltrating CD4+ TILs vs draining lymph node vs acute LCMV CD44+ T cells) by correlating fold-change (FC) vectors of cluster markers and retaining statistically robust cluster-pair matches.

## When to use

Use when two or more independently clustered scRNAseq datasets (e.g., replicates, tissues, or disease models like TIL vs dLN vs Arm CD4+ T cells) must be compared to find which clusters share transcriptomic profiles and which remain distinct.

## When NOT to use

- Datasets have not yet been individually clustered — the method requires precomputed per-dataset clusters and marker fold-change statistics as input
- The goal is de novo joint clustering on a single pooled matrix rather than matching independently derived clusters across pre-clustered datasets
- Only one dataset (no replicate or cross-condition comparison) is available, leaving no second set of clusters to correlate against

## Inputs

- Two or more independently clustered robustSingleCell analysis folders (e.g., preclustered LCMV1 and LCMV2 replicates of CD44+ T cell 10X data)
- Precomputed cluster assignments and differential-expression statistics per dataset
- Pooled project environment from initialize.project + read.preclustered.datasets

## Outputs

- cluster.similarity object with similarity matrix and map of cross-dataset cluster pairs
- filtered.similarity table of statistically robust cluster-pair matches (min.sd = qnorm(.9), max.q.val = 0.01)
- Robust cluster list and correlation heatmaps / similarity-statistics plots (paper Figures 9 and 10)
- Meta-cluster structure indicating shared vs distinct populations across datasets

## How to apply

First cluster each dataset separately with robustSingleCell to account for dataset-specific technical and biological differences, then initialize a pooled project with initialize.project and load the precomputed analyses with read.preclustered.datasets. Compute fold-change (FC) vectors for every cluster in each dataset using assess.cluster.similarity, and quantify similarity between all cluster pairs via Pearson correlation of the FC vectors. Filter cluster-pair similarities with get.robust.cluster.similarity using min.sd = qnorm(.9) and max.q.val = 0.01 so only robust, reproducible cross-replicate matches survive. Visualize matched clusters' FC-vector correlations as heatmaps (visualize.cluster.cors.heatmaps) and overall similarity statistics (visualize.cluster.similarity.stats) to recover the meta-cluster structure linking corresponding populations; in the paper this recovered the finding that tumor-specific CD4+ TIL profiles are largely distinct from dLN and acute virus-responsive (Arm) CD4+ T cells.

## Related tools

- **robustSingleCell** (Provides assess.cluster.similarity, get.robust.cluster.similarity, visualize.cluster.cors.heatmaps, and visualize.cluster.similarity.stats used to compute and filter cross-dataset cluster correlations) — github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell pipeline and its correlation analysis are executed)

## Examples

```
cluster.similarity <- assess.cluster.similarity(pooled_env); filtered.similarity <- get.robust.cluster.similarity(pooled_env, similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F)
```

## Evaluation signals

- Only cluster pairs passing min.sd = qnorm(.9) and max.q.val = 0.01 are reported as robust matches
- Generated correlation heatmaps and similarity-statistics figures reproduce the meta-cluster structure of paper Figures 9 and 10
- Each robust cluster pair's FC vectors show high Pearson correlation across replicates
- Biological sanity check: matched clusters map onto expected populations (e.g., Tfh, Th1, Tcmp, Treg marker sets used by get.cluster.names), and TIL clusters remain distinct from dLN/Arm clusters
- Clustering resolutions chosen per dataset show more than 2-fold modularity difference relative to shuffled-data analysis before integration

## Limitations

- The robustSingleCell package is not currently maintained, per the README
- Datasets must be clustered separately first to account for dataset-specific technical and biological differences; the method cannot be run directly on raw matrices
- The tutorial pipeline requires at least 8G of memory on a slurm high-performance computing workload manager
- Findings are correlative: robust cluster matching does not itself prove functional equivalence across tissues or models

## Evidence

- [methods] filtered.similarity <- get.robust.cluster.similarity: "filtered.similarity <- get.robust.cluster.similarity(
  pooled_env, similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F"
- [methods] assess.cluster.similarity call: "cluster.similarity <- assess.cluster.similarity(pooled_env)"
- [readme] separate clustering rationale: "We cluster each dataset separately to account for dataset-specific technical and biological differences."
- [readme] correlation analysis of clusters: "Then, we measure the transcriptional similarity and divergence between the clusters identified in the two datasets using correlation analysis."
- [readme] package maintenance status: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
- [abstract] TIL distinctness finding: "identify TIL transcriptomic patterns strikingly distinct from acute and chronic anti-viral responses and dominated by diversity among T-bet-expressing T helper type 1 (Th1)-like cells"
