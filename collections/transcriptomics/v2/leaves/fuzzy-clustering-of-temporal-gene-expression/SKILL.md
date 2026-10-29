---
name: fuzzy-clustering-of-temporal-gene-expression
description: Use when you have L-R pair (or gene) expression changes across ordered
  developmental time points and need to group them into temporal expression clusters
  and identify the peak time point of each cluster.
when_to_use_negative:
- The expression data are from a single stationary condition rather than an ordered
  developmental time series — dsCellNet's DTW and fuzzy clustering leverage developmental
  series information and do not apply.
- You need a hard (crisp) partition of genes into clusters; fuzzy c-means returns
  graded membership values requiring a membership threshold (e.g., 0.6) that may be
  inappropriate for binary assignments.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Mfuzz
  role: Performs the fuzzy c-means clustering of standardized L-R pair expression
    changes into eight clusters
- name: dsCellNet
  role: Loads and prepares the developmental-series L-R pair expression matrix via
    the datainput function before clustering
  repo: https://github.com/songzh523/dsCellNet
- name: R
  role: Implementation environment; dsCellNet is an open-access R package and Mfuzz
    is an R/Bioconductor package
  repo: https://github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:c167ed60abda19206f13b6d59a182617fbbf032d3f3c5540f5ec83c808548d30
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - Mfuzz
  - R
  - dsCellNet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fuzzy-clustering-of-temporal-gene-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Mfuzz fuzzy c-means clustering to standardized developmental-series expression profiles of ligand-receptor pair genes (e.g., from the GSE153164 corticogenesis dataset processed with dsCellNet) to assign interactions into eight temporal clusters and identify key developmental time points.

## When to use

Use when you have L-R pair (or gene) expression changes across ordered developmental time points and need to group them into temporal expression clusters and identify the peak time point of each cluster.

## When NOT to use

- The expression data are from a single stationary condition rather than an ordered developmental time series — dsCellNet's DTW and fuzzy clustering leverage developmental series information and do not apply.
- You need a hard (crisp) partition of genes into clusters; fuzzy c-means returns graded membership values requiring a membership threshold (e.g., 0.6) that may be inappropriate for binary assignments.

## Inputs

- Developmental-series expression matrix of retained ligand-receptor (L-R) pair genes across time points (from dsCellNet output on GSE153164 corticogenesis data)
- Cell-type label file and time-point file consumed by dsCellNet's datainput function
- Standardized (mean 0, SD 1) temporal expression-change matrix formatted for Mfuzz

## Outputs

- Cluster membership table (gene, cluster id, membership value), e.g. mfuzz_cluster_membership.csv
- Eight temporal clusters with identified peak time points (e.g., cluster 1 peaking at E12.5, cluster 4 at E15.5, cluster 5 at E17.5)

## How to apply

Load the developmental-series expression matrix of retained L-R pairs using the dsCellNet datainput function with expression, cell-type, and time-point files. Standardize the L-R pair expression changes across time points to mean 0 and standard deviation 1, as required by Mfuzz. Run Mfuzz fuzzy c-means clustering on the standardized temporal profiles with the default of eight clusters, as dsCellNet clusters interactions into eight distinct clusters. Filter cluster assignments at a membership >= 0.6 threshold to retain core members of each cluster, and write the membership table (gene, cluster id, membership value) to file. Rationale: fuzzy clustering in dsCellNet refines key time points and connections of developmental-series cell-cell interactions.

## Related tools

- **Mfuzz** (Performs the fuzzy c-means clustering of standardized L-R pair expression changes into eight clusters)
- **dsCellNet** (Loads and prepares the developmental-series L-R pair expression matrix via the datainput function before clustering) — https://github.com/songzh523/dsCellNet
- **R** (Implementation environment; dsCellNet is an open-access R package and Mfuzz is an R/Bioconductor package) — https://github.com/songzh523/dsCellNet

## Examples

```
cluster <- mfuzz(eset, c = 8, m = mestimate(eset))  # eset: ExpressionSet of standardized L-R pair temporal profiles
```

## Evaluation signals

- The membership table contains eight distinct clusters, matching the default Mfuzz cluster count used in dsCellNet.
- The largest peak clusters reproduce the reported sizes: cluster 1 at E12.5 with 564 genes, cluster 4 at E15.5 with 407 genes, and cluster 5 at E17.5 with 306 genes (membership >= 0.6).
- Each retained cluster member has a membership value >= 0.6 in the filtered table.
- Standardized input profiles have mean 0 and standard deviation 1 across time points before clustering.
- Each cluster's peak time point aligns with one of the GSE153164 corticogenesis time points (e.g., E12.5, E15.5, E17.5).

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); users can supply their own L-R genes as supplementary support.
- Fuzzy clustering requires a membership threshold (e.g., 0.6) to define core members, and the article does not state a dsCellNet version number, so exact package version reproducibility is uncertain.
- Cluster assignments depend on the standardization of expression changes; unstandardized or non-temporal data will not cluster meaningfully.

## Evidence
