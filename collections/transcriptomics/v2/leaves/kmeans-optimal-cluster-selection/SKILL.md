---
name: kmeans-optimal-cluster-selection
description: Use when simulated or processed count-based gene profiles have known
  ground-truth co-expression/temporal patterns and you need to verify that unsupervised
  clustering recovers them.
when_to_use_negative:
- Input is already a feature table without count-level information, so CPM > 1 filtering
  and log(x+1) transformation cannot be applied
- There is no ground-truth pattern assignment, so the incorrect cluster-assignment
  rate cannot be computed as validation
- Profiling single-cell data, where the article instead uses Spearman's distance and
  k-medoids clustering of cell-type-averaged profiles
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: MOSim
  role: Generates the simulated bulk RNA-seq (and ATAC-seq) count data with known
    temporal co-expression patterns used as clustering input and ground truth
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Environment for running MOSim simulation, CPM filtering, log transformation,
    k-means clustering, Silhouette selection, and PCA validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:406477d7503d974702ffdb0bceb237176f1278e302cf2ea799a9723f99b657aa
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kmeans-optimal-cluster-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies k-means clustering to simulated (or processed) gene feature profiles with the number of clusters chosen by maximizing the Silhouette coefficient and minimizing intra-cluster variability, then validates recovery of ground-truth temporal expression patterns via an incorrect cluster-assignment rate.

## When to use

Use when simulated or processed count-based gene profiles have known ground-truth co-expression/temporal patterns and you need to verify that unsupervised clustering recovers them.

## When NOT to use

- Input is already a feature table without count-level information, so CPM > 1 filtering and log(x+1) transformation cannot be applied
- There is no ground-truth pattern assignment, so the incorrect cluster-assignment rate cannot be computed as validation
- Profiling single-cell data, where the article instead uses Spearman's distance and k-medoids clustering of cell-type-averaged profiles

## Inputs

- Simulated RNA-seq count matrix from mosim() (bulk, e.g. STATegra-seeded, five time points, two groups, three replicates, 15% DEGs)
- Ground-truth simulated temporal expression pattern labels per gene

## Outputs

- k-means cluster assignments for gene profiles at the optimal k (e.g. k = 7)
- Incorrect cluster-assignment rate table saved to cluster_assessment.csv
- Optional PCA plot confirming clustering alignment with simulation settings

## How to apply

Filter lowly expressed features with a CPM > 1 threshold and apply a log(x+1) transformation to the simulated count data. Run k-means on the transformed gene feature profiles, selecting the optimal number of clusters by combining maximization of Silhouette's coefficient with minimization of intra-cluster variability (in the MOSim STATegra benchmark this yielded k = 7 for the five simulated temporal co-expression profiles). Compare the k-means cluster assignments against the ground-truth simulated temporal expression patterns, checking that each cluster corresponds to one pattern with its characteristic time point of maximal/minimal expression. Compute the incorrect cluster-assignment rate and save the result table (e.g. cluster_assessment.csv); a rate below 0.5% indicates correct simulation recovery. Optionally run PCA on the transformed data to confirm the clustering aligns with the simulation settings.

## Related tools

- **MOSim** (Generates the simulated bulk RNA-seq (and ATAC-seq) count data with known temporal co-expression patterns used as clustering input and ground truth) — https://github.com/ConesaLab/MOSim
- **R** (Environment for running MOSim simulation, CPM filtering, log transformation, k-means clustering, Silhouette selection, and PCA validation)

## Examples

```
Rscript run_bulk_mosim_kmeans_validation.R --seed STATegra --k 7 --cpm 1 --out cluster_assessment.csv
```

## Evaluation signals

- Incorrect cluster-assignment rate for recovering simulated temporal expression patterns is below 0.5%
- Each k-means cluster corresponds to exactly one simulated pattern with a distinct time point of maximal/minimal expression
- Optimal k from Silhouette maximization plus intra-cluster variability minimization matches the number of simulated patterns (k = 7 in the STATegra benchmark)
- PCA of the log(x+1)-transformed, CPM > 1-filtered data confirms clustering aligns with the simulation settings
- Result table cluster_assessment.csv is produced with per-gene cluster and ground-truth pattern assignments

## Limitations

- Validation threshold (<0.5% incorrect assignments) was established on the STATegra-seeded bulk benchmark; other datasets or designs may not meet it
- Choice of optimal k depends on the Silhouette/intra-cluster variability combination and may vary if the simulated design (e.g. number of temporal patterns) changes

## Evidence

- [methods] log transformation: [evidence span withheld — anchor status: not_found]
