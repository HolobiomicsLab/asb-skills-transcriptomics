---
name: clustering-accuracy-assessment
description: Use when simulated count data has known ground-truth co-expression or
  temporal pattern labels and you must quantify how faithfully a clustering method
  (k-means) recovers those patterns.
when_to_use_negative:
- Input is real experimental data with no ground-truth pattern labels — accuracy cannot
  be computed against simulated patterns, so this skill's core comparison is undefined
- 'Single-cell data: MOSim''s single-cell validation instead uses Spearman''s distance
  and k-medoids (partition around medoids) on cell-type-aggregated counts, not k-means
  on bulk gene profiles'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the bulk multi-omics count data (mosim function) with ground-truth
    temporal co-expression patterns used as clustering targets
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Runtime environment for MOSim, k-means clustering, Silhouette-based k selection,
    and PCA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:d26134dd1d2322af4f09d0d4ad6f207e320fdf40f894db9fe51bab157be5c904
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clustering-accuracy-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that a simulator (e.g., MOSim bulk simulation) has correctly reproduced temporal co-expression patterns by k-means clustering simulated gene profiles and comparing cluster assignments against the ground-truth pattern labels, reporting an incorrect cluster-assignment rate.

## When to use

Use when simulated count data has known ground-truth co-expression or temporal pattern labels and you must quantify how faithfully a clustering method (k-means) recovers those patterns.

## When NOT to use

- Input is real experimental data with no ground-truth pattern labels — accuracy cannot be computed against simulated patterns, so this skill's core comparison is undefined
- Single-cell data: MOSim's single-cell validation instead uses Spearman's distance and k-medoids (partition around medoids) on cell-type-aggregated counts, not k-means on bulk gene profiles

## Inputs

- Simulated bulk RNA-seq (or multi-omics) count matrix from MOSim with known per-gene ground-truth temporal co-expression pattern labels
- Ground-truth pattern/time-point assignment table from the simulation design (e.g., five temporal profiles, five time points, two groups, three replicates)

## Outputs

- K-means cluster assignment per gene profile for the optimal k
- Incorrect cluster-assignment rate (fraction of profiles misassigned relative to ground truth)
- cluster_assessment.csv result table
- PCA plot confirming clustering alignment with simulation settings

## How to apply

First filter lowly expressed genes with a CPM > 1 threshold and apply a log(x+1) transformation to the simulated count matrix. Run k-means on the transformed gene feature profiles with the optimal number of clusters k (in the MOSim STATegra validation, k = 7) selected by maximizing the Silhouette coefficient while minimizing intra-cluster variability. Compare the k-means assignments against the ground-truth simulated temporal expression patterns (five profiles across five time points in the reference case) and compute the incorrect cluster-assignment rate, saving the result table (e.g., cluster_assessment.csv). Run PCA on the same transformed data to confirm that clustering aligns with the simulation settings; the validation criterion in the MOSim paper is an incorrect-assignment rate below 0.5%.

## Related tools

- **MOSim** (Generates the bulk multi-omics count data (mosim function) with ground-truth temporal co-expression patterns used as clustering targets) — github.com/ConesaLab/MOSim
- **R** (Runtime environment for MOSim, k-means clustering, Silhouette-based k selection, and PCA)

## Examples

```
km <- kmeans(t(log(sim_counts + 1)), centers = 7); misassigned <- mean(km$cluster != ground_truth_patterns)
```

## Evaluation signals

- Incorrect cluster-assignment rate (misassigned gene profiles / total) is below 0.5%
- With k = 7, each simulated pattern yields exactly one cluster corresponding to its time point of maximal/minimal expression
- Optimal k was chosen by maximizing the Silhouette coefficient and minimizing intra-cluster variability
- PCA on the log(x+1)-transformed, CPM > 1-filtered data confirms clusters align with the simulation settings
- Result table written to cluster_assessment.csv with per-gene cluster vs. ground-truth pattern assignments

## Limitations

- Accuracy is only meaningful for simulation studies where ground-truth temporal patterns are known; it cannot validate real data
- Cluster validation depends on the k-selection criterion (Silhouette + intra-cluster variability); a suboptimal k can inflate the misassignment rate
- MOSim's underlying simulation does not extend to proteomics/metabolomics or spatial GRNs, so this validation applies only to the omics layers MOSim simulates

## Evidence
