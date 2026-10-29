---
name: temporal-expression-profile-clustering
description: Use when simulated (or otherwise ground-truth-labeled) bulk time-course
  gene count data must be checked for faithful recovery of its assigned temporal expression
  patterns via unsupervised clustering.
when_to_use_negative:
- 'Single-cell simulated data: the article''s scRNA-seq validation instead used Spearman''s
  distance (1 − Spearman''s correlation) with k-medoids partitioning around medoids,
  not k-means'
- 'Clustering real data with no ground-truth temporal labels: the <0.5% incorrect-assignment
  validation requires known simulated patterns to compare against'
- Non-count omics data (e.g., proteomics or metabolomics), which the article explicitly
  lists as outside MOSim's applicability
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated bulk RNA-seq (plus ATAC-seq) time-course counts and
    ground-truth temporal co-expression profiles that are then clustered and validated
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Execution environment for mosim simulation, CPM filtering, log(x+1) transformation,
    k-means clustering, silhouette selection, and PCA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:f9595ade92e84ea82520363bbf2d93764bcc1c2813e5e30acf8edffd1b6f9e52
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# temporal-expression-profile-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that simulated bulk RNA-seq time-course data reproduce their assigned temporal co-expression patterns by k-means clustering the gene profiles and comparing assignments against simulation ground truth (in MOSim's case, k = 7 with a <0.5% incorrect-assignment acceptance threshold).

## When to use

Use when simulated (or otherwise ground-truth-labeled) bulk time-course gene count data must be checked for faithful recovery of its assigned temporal expression patterns via unsupervised clustering.

## When NOT to use

- Single-cell simulated data: the article's scRNA-seq validation instead used Spearman's distance (1 − Spearman's correlation) with k-medoids partitioning around medoids, not k-means
- Clustering real data with no ground-truth temporal labels: the <0.5% incorrect-assignment validation requires known simulated patterns to compare against
- Non-count omics data (e.g., proteomics or metabolomics), which the article explicitly lists as outside MOSim's applicability

## Inputs

- Simulated bulk RNA-seq count matrix (mosim output) with ground-truth temporal pattern labels per gene
- Simulation design parameters (time points, groups, replicates, DEG fraction, co-expression profiles)
- Optional accompanying ATAC-seq layer from the same mosim run

## Outputs

- cluster_assessment.csv with per-gene k-means cluster assignments and the incorrect cluster-assignment rate
- Optimal cluster number k (e.g., k = 7) selected by Silhouette coefficient and intra-cluster variability
- PCA plot confirming cluster alignment with simulation settings

## How to apply

Run the bulk simulation (mosim with RNA-seq and ATAC-seq layers, five time points, two groups, three biological replicates, 15% DEGs, five temporal co-expression profiles) and record the ground-truth pattern assigned to each gene. Filter lowly expressed genes at CPM > 1 and apply a log(x+1) transformation to the count matrix. Apply k-means clustering [39] to the transformed gene profiles with k set by combining maximization of Silhouette's coefficient and minimization of intra-cluster variability (k = 7 for the five simulated patterns plus time points of maximal/minimal expression). Confirm clustering aligns with the simulation settings using PCA, then compare each gene's cluster assignment to its simulated temporal pattern and compute the incorrect cluster-assignment rate, saving the table to cluster_assessment.csv. The rationale is that synthetic data with ground truth let you define true positives/negatives for benchmarking clustering on time-course multi-omics data.

## Related tools

- **MOSim** (Generates the simulated bulk RNA-seq (plus ATAC-seq) time-course counts and ground-truth temporal co-expression profiles that are then clustered and validated) — https://github.com/ConesaLab/MOSim
- **R** (Execution environment for mosim simulation, CPM filtering, log(x+1) transformation, k-means clustering, silhouette selection, and PCA)

## Examples

```
km <- kmeans(t(log(expr_cpm_filtered + 1)), centers = 7, nstart = 25); rate <- mean(km$cluster != truth_patterns)
```

## Evaluation signals

- Incorrect cluster-assignment rate (genes whose k-means cluster differs from the simulated temporal pattern) must be below 0.5%
- Optimal k (selected by maximizing Silhouette's coefficient and minimizing intra-cluster variability) equals 7, giving one cluster per simulated pattern and time point of maximal/minimal expression
- PCA on the log(x+1)-transformed, CPM>1-filtered data confirms clusters align with the simulation settings (five time points, two groups)
- Each of the five simulated temporal co-expression profiles is recovered by exactly one k-means cluster

## Limitations

- k-means cluster count must be pre-selected via silhouette/intra-cluster variability trade-off; a misspecified k splits or merges temporal patterns and inflates the incorrect-assignment rate
- Validation is only meaningful on data with simulated ground-truth pattern labels; on real data the <0.5% criterion cannot be computed
- The clustering-based validation as described applies to bulk profiles; single-cell profiles in the same paper required a different approach (k-medoids with Spearman's distance on cell-type-aggregated counts)

## Evidence

- [methods] log transformation: [evidence span withheld — anchor status: not_found]
