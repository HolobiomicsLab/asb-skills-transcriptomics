---
name: cluster-membership-table-export
description: Use when you have a candidate L-R pair expression-change matrix from
  dsCellNet's developcorrelation DTW step for a developmental series (e.g. GSE153164
  corticogenesis) and need to partition L-R pairs into time-point-specific clusters
  and write out per-cluster gene lists.
when_to_use_negative:
- Input is a single-time-point expression dataset with no developmental series time
  information, since DTW-based expression changes and temporal cluster assignment
  are not applicable
- You need a single static cell-cell interaction map rather than time-resolved clustering
  of L-R interactions; dsCellNet is designed for developmental series RNA-seq
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dsCellNet
  role: Upstream pipeline (preparepairs, developcorrelation) producing the L-R pair
    expression-change matrix that is clustered and exported
  repo: github.com/songzh523/dsCellNet
- name: Mfuzz
  role: Fuzzy c-means clustering (mestimate fuzzifier, min.std pre-filter, eight clusters)
    of the L-R expression-change matrix and generation of the membership table
- name: R
  role: Implementation environment; dsCellNet is an open-access R package and Mfuzz
    is an R/Bioconductor package
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:8a5e47eb0956ce2542892af9b5e86fa0d94a2d211d2c02d8b78f6e28931f7a3a
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cluster-membership-table-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Mfuzz fuzzy c-means clustering on the dsCellNet developmental-series L-R interaction expression-change matrix (eight clusters, fuzzifier estimated with mestimate) and export the full membership/assignment table, e.g. mfuzz_cluster_assignments.csv, so that downstream GO enrichment can be run on per-cluster gene lists such as the E12.5 cluster 1 L-R genes.

## When to use

Use when you have a candidate L-R pair expression-change matrix from dsCellNet's developcorrelation DTW step for a developmental series (e.g. GSE153164 corticogenesis) and need to partition L-R pairs into time-point-specific clusters and write out per-cluster gene lists.

## When NOT to use

- Input is a single-time-point expression dataset with no developmental series time information, since DTW-based expression changes and temporal cluster assignment are not applicable
- You need a single static cell-cell interaction map rather than time-resolved clustering of L-R interactions; dsCellNet is designed for developmental series RNA-seq

## Inputs

- Interaction expression-change matrix for candidate L-R pairs from dsCellNet developcorrelation DTW correlation of a developmental series scRNA-seq dataset (e.g. GSE153164 corticogenesis time points)
- Number of clusters (eight, per dsCellNet)
- Fuzzifier m estimated by Mfuzz mestimate and min.std pre-filtering threshold

## Outputs

- mfuzz_cluster_assignments.csv: table of per-L-R-pair fuzzy memberships and maximum-membership cluster assignments across eight clusters
- Per-cluster L-R gene lists (e.g. cluster 1 genes highly expressed at E12.5) for downstream GO enrichment

## How to apply

Load the interaction expression-change matrix computed from the GSE153164 corticogenesis developmental series (upstream output of dsCellNet's preparepairs/developcorrelation DTW correlation step). Standardize the expression-change matrix and pre-filter low-variance L-R pairs using Mfuzz's min.std criterion, as described for clustering L-R pair expression changes. Run Mfuzz fuzzy c-means with eight clusters, estimating the fuzzifier m via mestimate, mirroring the article's clustering of L-R expression changes into eight distinct clusters. Assign each L-R pair to its maximum-membership cluster and write the complete membership/assignment table (including the cluster 1 gene list of E12.5-high genes) to mfuzz_cluster_assignments.csv for downstream GO enrichment of e.g. stem-cell pluripotency functions.

## Related tools

- **dsCellNet** (Upstream pipeline (preparepairs, developcorrelation) producing the L-R pair expression-change matrix that is clustered and exported) — github.com/songzh523/dsCellNet
- **Mfuzz** (Fuzzy c-means clustering (mestimate fuzzifier, min.std pre-filter, eight clusters) of the L-R expression-change matrix and generation of the membership table)
- **R** (Implementation environment; dsCellNet is an open-access R package and Mfuzz is an R/Bioconductor package) — github.com/songzh523/dsCellNet

## Examples

```
developcorrelation(inptim,usedcolo,resloc)
```

## Evaluation signals

- The exported mfuzz_cluster_assignments.csv partitions the L-R pairs into exactly eight distinct clusters
- Every candidate L-R pair appears in the table and is assigned to its maximum-membership cluster (assignment invariant)
- Row count of the output table equals the number of L-R pairs surviving the Mfuzz min.std variance pre-filter
- Cluster 1 gene list corresponds to interactions peaking at E12.5, consistent with GO enrichment for stem cell proliferation, division, and stem cell population maintenance functions
- Fuzzifier m used in clustering equals the value returned by mestimate for the standardized matrix

## Limitations

- The default L-R gene list size limits detection of some interactions; users can supplement their own L-R gene list if expected pairs are missing
- Fuzzy clustering helps identify the key time point and mediator cell types but does not by itself validate biological function; GO enrichment is a separate downstream step
- Low-variance L-R pairs removed by the min.std pre-filter will be absent from the membership table

## Evidence

- [other] Standardize the expression-change matrix and filter low-variance L-R pairs using Mfuzz's min.std pre-filtering.: [evidence span withheld — anchor status: not_found]
- [other] Run Mfuzz fuzzy c-means clustering on the filtered matrix, setting the number of clusters to eight and estimating the fuzzifier m with mestimate: [evidence span withheld — anchor status: not_found]
