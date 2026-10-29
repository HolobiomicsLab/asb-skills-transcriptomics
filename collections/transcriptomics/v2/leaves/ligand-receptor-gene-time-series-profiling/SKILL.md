---
name: ligand-receptor-gene-time-series-profiling
description: Use when you have expression values for retained L-R pairs across ordered
  developmental time points (e.g., GSE153164 E12.5–E17.5 corticogenesis) and need
  to identify which interactions peak at which stage.
when_to_use_negative:
- Data are from a single stationary condition with no developmental-series time points
  — dsCellNet's DTW and fuzzy clustering leverage temporal information and a single
  time-point map cannot capture dynamic interaction changes
- Input is a cell-cell interaction network table rather than per-time-point gene expression
  of L-R pairs — clustering operates on standardized expression changes of the L-R
  pairs, not on network edges
- You only need a single-time-point cell-cell interaction network; use dsCellNet's
  singlenet workflow instead of the developmental clustering step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dsCellNet
  role: Loads the developmental-series expression, cell-type, and time-point files
    (datainput) and supplies the retained L-R pair expression used for temporal clustering
  repo: https://github.com/songzh523/dsCellNet
- name: Mfuzz
  role: Performs fuzzy c-means clustering of the standardized L-R pair temporal expression
    profiles into eight clusters with a membership >= 0.6 core-member cutoff
- name: R
  role: Implementation environment; dsCellNet is an open-access R package requiring
    R >= 4.0.3
  repo: https://github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:b4665a0b8b44b19519923c2461efabf25c487646f7f8a91dbdca52ed6ac4cfc0
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ligand-receptor-gene-time-series-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles the temporal expression trajectories of ligand-receptor (L-R) pair genes across developmental-series time points and groups them with Mfuzz fuzzy c-means clustering into temporal clusters that pinpoint key time points of interaction activity during corticogenesis.

## When to use

Use when you have expression values for retained L-R pairs across ordered developmental time points (e.g., GSE153164 E12.5–E17.5 corticogenesis) and need to identify which interactions peak at which stage.

## When NOT to use

- Data are from a single stationary condition with no developmental-series time points — dsCellNet's DTW and fuzzy clustering leverage temporal information and a single time-point map cannot capture dynamic interaction changes
- Input is a cell-cell interaction network table rather than per-time-point gene expression of L-R pairs — clustering operates on standardized expression changes of the L-R pairs, not on network edges
- You only need a single-time-point cell-cell interaction network; use dsCellNet's singlenet workflow instead of the developmental clustering step

## Inputs

- Developmental-series expression matrix of retained L-R pairs (expression file)
- Cell-type label file
- Time-point file (e.g., E12.5, E14.5, E15.5, E17.5 from GSE153164 corticogenesis series)

## Outputs

- mfuzz_cluster_membership.csv (gene, cluster id, membership value) with core members at membership >= 0.6
- Eight temporal clusters with per-cluster peak time points and member gene counts

## How to apply

Load the developmental-series expression matrix of retained L-R pairs via the dsCellNet datainput function, supplying the expression, cell-type, and time-point files. Standardize the L-R pair expression changes across time points to mean 0 and standard deviation 1, as required by Mfuzz. Run Mfuzz fuzzy c-means clustering on the standardized temporal profiles with the default of eight clusters. Filter cluster assignments at a membership >= 0.6 threshold to keep only core members of each cluster, then write the gene–cluster–membership table (mfuzz_cluster_membership.csv) and check whether the largest peak clusters align with the reported E12.5-peak (cluster 1, 564 genes), E15.5-peak (cluster 4, 407 genes), and E17.5-peak (cluster 5, 306 genes) clusters.

## Related tools

- **dsCellNet** (Loads the developmental-series expression, cell-type, and time-point files (datainput) and supplies the retained L-R pair expression used for temporal clustering) — https://github.com/songzh523/dsCellNet
- **Mfuzz** (Performs fuzzy c-means clustering of the standardized L-R pair temporal expression profiles into eight clusters with a membership >= 0.6 core-member cutoff)
- **R** (Implementation environment; dsCellNet is an open-access R package requiring R >= 4.0.3) — https://github.com/songzh523/dsCellNet

## Examples

```
eset <- standardise(exprs); cl <- mfuzz(eset, c = 8, min.mem = 0.6); write.csv(cl$cluster, "mfuzz_cluster_membership.csv")
```

## Evaluation signals

- The membership table contains exactly eight distinct clusters (the Mfuzz default)
- Cluster 1 peaks at E12.5 with 564 genes, cluster 4 at E15.5 with 407 genes, and cluster 5 at E17.5 with 306 genes, matching the reported cluster sizes
- Every retained cluster member has a membership value >= 0.6 after filtering
- Expression changes of L-R pairs were standardized to mean 0 and standard deviation 1 before clustering, as required by Mfuzz

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); users can supply their own L-R gene list as a supplement
- Fuzzy clustering memberships below 0.6 are discarded, so genes with ambiguous temporal profiles are excluded from core cluster assignments
- Reproduction depends on matching the reported peak-cluster numbering (clusters 1, 4, 5), which may shift across Mfuzz runs or parameter changes

## Evidence

- [other] Membership >= 0.6 threshold and expected cluster sizes: [evidence span withheld — anchor status: not_found]
