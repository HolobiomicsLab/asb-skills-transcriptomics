---
name: statistical-filtering-of-cluster-pairs
description: 'Use when cross-replicate or cross-dataset scRNAseq clusters must be
  matched: after assess.cluster.similarity computes FC-vector correlations, to discard
  non-reproducible cluster pairs before building a meta-cluster map.'
when_to_use_negative:
- Datasets have not yet been individually clustered and summarized — filtering requires
  precomputed per-dataset cluster analyses loaded via read.preclustered.datasets
- Only one replicate/dataset is available — cross-replicate correlation robustness
  cannot be assessed, so the min.sd threshold is undefined
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Provides assess.cluster.similarity and get.robust.cluster.similarity functions
    that compute FC-vector Pearson correlations and apply the min.sd = qnorm(.9),
    max.q.val = 0.01 statistical filter
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell pipeline and the qnorm-based
    filtering call are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:350b681b9ecf392d774be35a09e5599c50b643a692d7ef313b7a7f9fc4c21c06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# statistical-filtering-of-cluster-pairs

## Summary

A robustSingleCell step that filters pairwise cross-dataset cluster similarities so only statistically robust, reproducible cluster matches are retained, using fold-change (FC) vector Pearson correlations across replicates and significance thresholds.

## When to use

Use when cross-replicate or cross-dataset scRNAseq clusters must be matched: after assess.cluster.similarity computes FC-vector correlations, to discard non-reproducible cluster pairs before building a meta-cluster map.

## When NOT to use

- Datasets have not yet been individually clustered and summarized — filtering requires precomputed per-dataset cluster analyses loaded via read.preclustered.datasets
- Only one replicate/dataset is available — cross-replicate correlation robustness cannot be assessed, so the min.sd threshold is undefined

## Inputs

- Pooled robustSingleCell project environment (e.g. pooled_env) initialized over independently clustered datasets such as LCMV1/LCMV2 or TIL/dLN/Arm
- Similarity object returned by assess.cluster.similarity (cluster.similarity$similarity), containing cross-replicate Pearson correlations of cluster FC vectors

## Outputs

- Filtered similarity data frame (filtered.similarity) of robust cluster pairs passing min.sd and q-value thresholds
- Vector of robust cluster identifiers (robust.clusters) for downstream heatmap and meta-cluster visualization

## How to apply

After clustering each dataset independently and pooling them with initialize.project and read.preclustered.datasets, run assess.cluster.similarity to compute fold-change (FC) vectors for every cluster in each dataset and their Pearson correlations. Pass the resulting similarity object to get.robust.cluster.similarity on the pooled environment with min.sd = qnorm(.9) and max.q.val = 0.01; this retains only cluster pairs whose correlation is robust across replicates and statistically significant after multiple-testing control. Collect the surviving endpoints via sort(unique(c(filtered.similarity$cluster1, filtered.similarity$cluster2))) to define robust clusters, then visualize with visualize.cluster.cors.heatmaps and visualize.cluster.similarity.stats. The rationale is to prevent spurious matches from driving the integrated TIL/dLN/Arm meta-cluster structure reported in Figures 9 and 10.

## Related tools

- **robustSingleCell** (Provides assess.cluster.similarity and get.robust.cluster.similarity functions that compute FC-vector Pearson correlations and apply the min.sd = qnorm(.9), max.q.val = 0.01 statistical filter) — github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell pipeline and the qnorm-based filtering call are executed)

## Examples

```
cluster.similarity <- assess.cluster.similarity(pooled_env); filtered.similarity <- get.robust.cluster.similarity(pooled_env, cluster.similarity$similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F)
```

## Evaluation signals

- Only cluster pairs passing min.sd = qnorm(.9) and max.q.val = 0.01 appear in filtered.similarity — no pair violating either threshold is retained
- The correlation heatmap from visualize.cluster.cors.heatmaps reproduces the structure of Figure 9 (correlations of matched clusters' FC vectors across replicates)
- visualize.cluster.similarity.stats recovers the meta-cluster structure linking corresponding populations as in Figure 10
- robust.clusters contains only endpoints of surviving pairs, computed as sort(unique(c(filtered.similarity$cluster1, filtered.similarity$cluster2)))

## Limitations

- robustSingleCell is not currently maintained; the README notes users must contact the authors to collaborate on improvements
- The pipeline supports slurm for parallel batch jobs and the tutorial analysis requires at least 8G of memory on a high performance computing workload manager
- Rerun behavior of the filter depends on the rerun argument (set to FALSE in the tutorial), which can reuse cached similarity results

## Evidence

- [methods] Filter cluster-pair similarities with statistically robust thresholds to retain reproducible cross-replicate cluster matches: "filtered.similarity <- get.robust.cluster.similarity(
  pooled_env, similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F"
- [readme] Robustness is defined by correlation of fold-change vectors across replicates: "Then, we measure the
transcriptional similarity and divergence between the clusters
identified in the two datasets using correlation analysis."
- [intro] Similarity analysis compares population compositions across tissues and models: "robustSingleCell is a pipeline designed to identify robust cell
subpopulations using scRNAseq data"
- [readme] Downstream visualization validates the filtered similarity structure: "Figure 9 shows
the correlation between clusters’ FC vectors across replicates (as
described in *Magen et al 2019*)."
- [intro] The package maintenance status is a documented limitation: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
