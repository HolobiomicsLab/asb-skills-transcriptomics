---
name: cell-cell-interaction-expression-profiling
description: Use when you have developmental series scRNA-seq (e.g., GSE153164 corticogenesis)
  with inferred L-R pair expression-change values and need to group interactions by
  temporal activity to pinpoint key developmental time points.
when_to_use_negative:
- The data are from a single stationary condition with no time-point information,
  since dsCellNet's fuzzy clustering leverages developmental series dynamics
- The goal is only to detect individual L-R pairs in one network (use singlenet),
  not to profile their expression changes across development
- Input is already a cluster assignment or enrichment table rather than a raw interaction
  expression-change matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: Upstream inference of L-R pairs and their expression-change values via preparepairs
    and developcorrelation (DTW) before fuzzy clustering
  repo: github.com/songzh523/dsCellNet
- name: Mfuzz
  role: Fuzzy c-means clustering of L-R pair expression changes into eight clusters,
    with min.std pre-filtering and mestimate fuzzifier
- name: R
  role: Implementation environment; dsCellNet is an open-access R package on GitHub
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:96defab4afee45ada5870d259c6e5fcea90e880db2fc7893110f3d12f4e32a53
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cell-cell-interaction-expression-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles the expression dynamics of candidate ligand–receptor (L-R) pairs across developmental series time points and partitions them into soft clusters using Mfuzz fuzzy c-means, so that key time points (e.g., E12.5 in corticogenesis) and mediator interactions can be identified.

## When to use

Use when you have developmental series scRNA-seq (e.g., GSE153164 corticogenesis) with inferred L-R pair expression-change values and need to group interactions by temporal activity to pinpoint key developmental time points.

## When NOT to use

- The data are from a single stationary condition with no time-point information, since dsCellNet's fuzzy clustering leverages developmental series dynamics
- The goal is only to detect individual L-R pairs in one network (use singlenet), not to profile their expression changes across development
- Input is already a cluster assignment or enrichment table rather than a raw interaction expression-change matrix

## Inputs

- Expression-change matrix of candidate L-R pairs across developmental series time points (from preparepairs/developcorrelation DTW step)
- Single-cell expression matrix with cell-type labels and time-point annotations (e.g., GSE153164 E12.5–E15.5 mouse embryonic cerebral cortex)

## Outputs

- Mfuzz cluster membership/assignment table for all L-R pairs (mfuzz_cluster_assignments.csv)
- Eight clusters of L-R pairs keyed to developmental time points, each amenable to downstream GO enrichment

## How to apply

Load the interaction expression-change matrix for candidate L-R pairs, computed upstream by dsCellNet's preparepairs/developcorrelation DTW correlation step from a developmental series such as GSE153164. Standardize the matrix and pre-filter low-variance L-R pairs using Mfuzz's min.std criterion. Run Mfuzz fuzzy c-means with the number of clusters set to eight and the fuzzifier m estimated with mestimate, following the dsCellNet approach of dividing candidate interactions into clusters. Assign each L-R pair to its highest-membership cluster and write the full membership/assignment table (e.g., mfuzz_cluster_assignments.csv). The rationale is that soft clustering captures gradual, time-resolved shifts in interaction strength, letting clusters be linked to specific stages (e.g., cluster 1 at E12.5 enriched for stem cell pluripotency functions).

## Related tools

- **dsCellNet** (Upstream inference of L-R pairs and their expression-change values via preparepairs and developcorrelation (DTW) before fuzzy clustering) — github.com/songzh523/dsCellNet
- **Mfuzz** (Fuzzy c-means clustering of L-R pair expression changes into eight clusters, with min.std pre-filtering and mestimate fuzzifier)
- **R** (Implementation environment; dsCellNet is an open-access R package on GitHub) — github.com/songzh523/dsCellNet

## Examples

```
mestimate(eset); cl <- mfuzz(eset, centers=8, m=m); memberships <- cl$membership
```

## Evaluation signals

- The output table partitions L-R pairs into exactly eight distinct clusters
- Every candidate L-R pair is assigned to its maximum-membership cluster (soft memberships summed per row = 1)
- Low-variance L-R pairs removed by Mfuzz min.std pre-filtering before clustering
- Fuzzifier m is obtained via mestimate rather than set arbitrarily
- Cluster 1 (E12.5) interactions reproduce enrichment for stem cell proliferation, division, and stem cell population maintenance GO terms

## Limitations

- Cluster membership is fuzzy, so assigning each pair to its single highest-membership cluster discards membership gradation
- The default L-R gene list size limits detection of some interactions; users can supply supplementary L-R genes
- Requires multi-time-point developmental series input; results depend on the quality of the upstream DTW correlation step

## Evidence

- [results] eight clusters via Mfuzz: "The expression changes of L-R pairs were clustered into eight distinct clusters using Mfuzz packages"
- [abstract] fuzzy clustering rationale: "Then, fuzzy clustering of those interactions helps us refine key time points and connections"
- [results] DTW then fuzzy clustering pipeline: "Finally, DTW and fuzzy clustering divided those candidate interactions into several clusters"
- [readme] upstream DTW correlation step: "developcorrelation(inptim,usedcolo,resloc)"
