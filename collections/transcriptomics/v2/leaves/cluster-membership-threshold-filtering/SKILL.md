---
name: cluster-membership-threshold-filtering
description: Use when temporal expression changes of candidate interactions (e.g.,
  L-R pairs) must be partitioned into soft clusters and only high-confidence core
  members of each cluster are needed to call peak time points.
when_to_use_negative:
- When only hard/unique cluster assignment is needed and soft membership values carry
  no downstream meaning — a standard k-means clustering would suffice instead of Mfuzz
  fuzzy c-means.
- When the expression data are from a single stationary condition rather than a developmental
  series — dynamic time warping and temporal fuzzy clustering in dsCellNet assume
  ordered time points.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Mfuzz
  role: Performs the fuzzy c-means clustering of standardized L-R pair expression
    changes into eight clusters, producing membership values that are filtered at
    >= 0.6
- name: dsCellNet
  role: Provides the datainput function to load the developmental-series expression,
    cell-type, and time-point files and supplies the retained L-R pair expression
    matrix that is clustered
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Language in which dsCellNet is implemented as an open-access R package and
    in which the Mfuzz clustering and membership filtering are executed
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:67599fa1d4af89f2b574f1f3ca34b7a02e0dbd66f2a6c1de1d543ad2f2c31ee9
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cluster-membership-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply fuzzy c-means clustering (Mfuzz) to standardized developmental-series expression profiles of ligand-receptor pairs and retain only core cluster members at a membership >= 0.6 threshold, as used in dsCellNet to resolve temporal clusters across GSE153164 corticogenesis time points.

## When to use

Use when temporal expression changes of candidate interactions (e.g., L-R pairs) must be partitioned into soft clusters and only high-confidence core members of each cluster are needed to call peak time points.

## When NOT to use

- When only hard/unique cluster assignment is needed and soft membership values carry no downstream meaning — a standard k-means clustering would suffice instead of Mfuzz fuzzy c-means.
- When the expression data are from a single stationary condition rather than a developmental series — dynamic time warping and temporal fuzzy clustering in dsCellNet assume ordered time points.

## Inputs

- Developmental-series expression matrix of retained L-R pairs (from GSE153164 corticogenesis scRNA-seq data, E12.5-E17.5 time points)
- Cell-type label file
- Time-point information file

## Outputs

- mfuzz_cluster_membership.csv containing gene, cluster id, and membership value for core members (membership >= 0.6)
- Eight distinct fuzzy clusters with identified peak time points (cluster 1 peaking at E12.5, cluster 4 at E15.5, cluster 5 at E17.5)

## How to apply

Load the developmental-series expression matrix of retained L-R pairs from the GSE153164 corticogenesis dataset using the dsCellNet datainput function with expression, cell-type, and time-point files. Standardize the expression changes of L-R pairs across time points to mean 0 and standard deviation 1, as required by Mfuzz. Run Mfuzz fuzzy c-means clustering with the default of eight clusters on the standardized temporal profiles. Filter cluster assignments at the membership >= 0.6 threshold to keep only core members of each cluster, then write a membership table (gene, cluster id, membership value) to mfuzz_cluster_membership.csv. The soft-membership filtering rationale is that fuzzy clustering refines key time points and connections, so low-membership genes are ambiguous and excluded from peak-cluster size counts.

## Related tools

- **Mfuzz** (Performs the fuzzy c-means clustering of standardized L-R pair expression changes into eight clusters, producing membership values that are filtered at >= 0.6)
- **dsCellNet** (Provides the datainput function to load the developmental-series expression, cell-type, and time-point files and supplies the retained L-R pair expression matrix that is clustered) — github.com/songzh523/dsCellNet
- **R** (Language in which dsCellNet is implemented as an open-access R package and in which the Mfuzz clustering and membership filtering are executed) — github.com/songzh523/dsCellNet

## Examples

```
mfuzz_res <- mfuzz(eset, c = 8, m = mestimate(eset)); core <- subset(cluster_table, membership >= 0.6); write.csv(core, "mfuzz_cluster_membership.csv")
```

## Evaluation signals

- The membership table must contain exactly eight distinct clusters, matching the Mfuzz default of eight clusters reported in the article.
- The largest peak clusters must reproduce the reported sizes: cluster 1 with 564 genes peaking at E12.5, cluster 4 with 407 genes at E15.5, and cluster 5 with 306 genes at E17.5.
- Every row in mfuzz_cluster_membership.csv must have membership >= 0.6; rows below this threshold must have been removed.
- Standardization invariance: each standardized L-R pair profile across time points should have mean ~0 and standard deviation ~1 before clustering.
- Each retained gene should show a coherent peak time point consistent with its assigned cluster's temporal profile.

## Limitations

- The size of the default L-R gene list (5,744 interactions / 5,684 in the comparison table) limits detection of some interactions, so clustering operates only on retained pairs.
- Low-membership genes (below 0.6) are discarded, which may exclude biologically relevant but temporally ambiguous interactions.
- The eight-cluster default may not be optimal for all developmental series; peak cluster reproducibility depends on the GSE153164 time-point sampling.

## Evidence
