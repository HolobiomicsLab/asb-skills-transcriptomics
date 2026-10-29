---
name: scrna-seq-developmental-trajectory-analysis
description: Use when analyzing how L-R pair expression changes across ordered developmental
  time points (e.g., GSE153164 corticogenesis) and you need to assign pairs to temporal
  clusters and identify peak time points.
when_to_use_negative:
- The data are a single stationary condition without a developmental time series —
  use standard single-time-point cell-cell communication inference instead (e.g.,
  singlenet step).
- You need per-time-point interaction networks rather than temporal grouping of L-R
  pair expression profiles — the DTW/comparednet steps are the appropriate dsCellNet
  stages.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Mfuzz
  role: Fuzzy c-means clustering of standardized L-R pair temporal expression profiles
    into eight clusters with membership >= 0.6 core filtering
- name: dsCellNet
  role: Host pipeline; datainput loads expression/cell-type/time-point files feeding
    the clustering, and fuzzy clustering refines key time points and mediator cell
    types
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Implementation environment; dsCellNet is an open-access R package and Mfuzz
    is an R/Bioconductor package
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:cdc610e76557400ac6f6f95c7695a1f804f0794ac1ccd1cc94b87dbdbe987256
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - Mfuzz
  - R
  - dsCellNet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scrna-seq-developmental-trajectory-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups developmental-series expression profiles of ligand-receptor pairs into temporal clusters using Mfuzz fuzzy c-means, refining key time points (e.g., E12.5, E15.5, E17.5) and mediator connections in the dsCellNet corticogenesis workflow.

## When to use

Use when analyzing how L-R pair expression changes across ordered developmental time points (e.g., GSE153164 corticogenesis) and you need to assign pairs to temporal clusters and identify peak time points.

## When NOT to use

- The data are a single stationary condition without a developmental time series — use standard single-time-point cell-cell communication inference instead (e.g., singlenet step).
- You need per-time-point interaction networks rather than temporal grouping of L-R pair expression profiles — the DTW/comparednet steps are the appropriate dsCellNet stages.

## Inputs

- Developmental-series expression matrix of retained L-R pairs from GSE153164 corticogenesis (expression, cell-type, and time-point files accepted by dsCellNet datainput)
- Time-point ordering (e.g., E12.5, E15.5, E17.5 ...)
- Standardized (mean 0, SD 1) temporal profile matrix of L-R pairs for Mfuzz

## Outputs

- mfuzz_cluster_membership.csv with columns gene, cluster id, membership value
- Eight temporal clusters with core members at membership >= 0.6
- Identification of peak time points per cluster (e.g., cluster 1 peaks at E12.5, cluster 4 at E15.5, cluster 5 at E17.5)

## How to apply

Load the developmental-series expression matrix of retained L-R pairs via the dsCellNet datainput function, supplying expression, cell-type, and time-point files. Standardize the L-R pair expression changes across time points to mean 0 and standard deviation 1 as required by Mfuzz. Run Mfuzz fuzzy c-means on the standardized temporal profiles with the default of eight clusters, then filter assignments at membership >= 0.6 to retain core members. Write the membership table (gene, cluster id, membership value) to mfuzz_cluster_membership.csv and check that peak clusters match reported ones (cluster 1 at E12.5 with 564 genes, cluster 4 at E15.5 with 407 genes, cluster 5 at E17.5 with 306 genes). This fuzzy clustering step is the dsCellNet stage that refines key time points and mediator cell types after DTW-based developmental correlation.

## Related tools

- **Mfuzz** (Fuzzy c-means clustering of standardized L-R pair temporal expression profiles into eight clusters with membership >= 0.6 core filtering)
- **dsCellNet** (Host pipeline; datainput loads expression/cell-type/time-point files feeding the clustering, and fuzzy clustering refines key time points and mediator cell types) — github.com/songzh523/dsCellNet
- **R** (Implementation environment; dsCellNet is an open-access R package and Mfuzz is an R/Bioconductor package) — github.com/songzh523/dsCellNet

## Examples

```
out <- datainput(datloc, expnam, typnam, timnam, "FALSE", resloc); inpexp <- as.data.frame(out$exp)
```

## Evaluation signals

- mfuzz_cluster_membership.csv contains exactly eight distinct clusters
- Cluster 1 (E12.5-peak) contains 564 genes at membership >= 0.6
- Cluster 4 (E15.5-peak) contains 407 genes at membership >= 0.6
- Cluster 5 (E17.5-peak) contains 306 genes at membership >= 0.6
- Every retained row has membership >= 0.6 and columns gene/cluster id/membership value

## Limitations

- Default L-R gene list size limits detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes.
- The dsCellNet version number is not stated in the article.
- Clustering depends on standardized expression changes of retained pairs; pairs dropped by upstream filtering are not represented.

## Evidence
