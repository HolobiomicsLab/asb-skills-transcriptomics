---
name: fuzzy-c-means-time-series-clustering
description: Use when you have a matrix of DTW-computed expression changes for L-R
  pairs across developmental time points and need to identify which interactions are
  active at a given stage (e.g., cluster 1 = E12.5 pluripotency-related interactions).
when_to_use_negative:
- Input is a single-time-point scRNA-seq dataset with no developmental series — dsCellNet's
  DTW + fuzzy clustering pipeline requires temporal series data
- The expression-change matrix has not been pre-filtered for variance (low-variance
  L-R pairs must be removed via Mfuzz's min.std before clustering)
- The goal is hard partitioning of stationary-condition L-R gene expression rather
  than dynamic interaction changes across development
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dsCellNet
  role: Upstream pipeline producing the DTW-based L-R interaction expression-change
    matrix that is clustered
  repo: github.com/songzh523/dsCellNet
- name: Mfuzz
  role: R package providing fuzzy c-means clustering, min.std pre-filtering, and mestimate
    fuzzifier estimation applied to the L-R expression-change matrix
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Implementation environment; dsCellNet is an open-access R package and Mfuzz
    clustering is run in R
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:6d3d2501cfc5952cbd8f765cf386ae68f788639a4f10711a8ddb95cec389071d
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Fuzzy c-means clustering of developmental-series L-R interaction expression changes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions expression-change trajectories of candidate ligand-receptor pairs from a developmental series into soft clusters using Mfuzz fuzzy c-means, so that interactions peaking at specific developmental time points (e.g., E12.5 during corticogenesis) can be isolated for downstream functional enrichment. Fuzzy membership, rather than hard assignment, captures partial co-regulation of interactions across time points.

## When to use

Use when you have a matrix of DTW-computed expression changes for L-R pairs across developmental time points and need to identify which interactions are active at a given stage (e.g., cluster 1 = E12.5 pluripotency-related interactions).

## When NOT to use

- Input is a single-time-point scRNA-seq dataset with no developmental series — dsCellNet's DTW + fuzzy clustering pipeline requires temporal series data
- The expression-change matrix has not been pre-filtered for variance (low-variance L-R pairs must be removed via Mfuzz's min.std before clustering)
- The goal is hard partitioning of stationary-condition L-R gene expression rather than dynamic interaction changes across development

## Inputs

- Interaction expression-change matrix for candidate L-R pairs across developmental time points (from dsCellNet developcorrelation DTW step)
- GSE153164 developmental-series scRNA-seq data (expression, cell-type labels, time-point files) as processed upstream by datainput/preparepairs/singlenet

## Outputs

- mfuzz_cluster_assignments.csv: table of L-R pair memberships and highest-membership cluster assignments across eight clusters
- Cluster 1 (E12.5) L-R gene list for downstream GO enrichment analysis

## How to apply

Load the interaction expression-change matrix for candidate L-R pairs computed upstream by dsCellNet's preparepairs/developcorrelation DTW correlation step on the GSE153164 corticogenesis series. Standardize the matrix and pre-filter low-variance L-R pairs using Mfuzz's min.std criterion. Run Mfuzz fuzzy c-means with the number of clusters set to eight (as dsCellNet clusters expression changes of L-R pairs into eight distinct clusters) and estimate the fuzzifier m with mestimate(). Assign each L-R pair to its highest-membership cluster and write the full membership/assignment table, including the cluster 1 (E12.5) gene list, to mfuzz_cluster_assignments.csv. The soft memberships let you recover key time points and mediator interactions, which can then feed GO enrichment (e.g., cluster 1 enrichment for stem cell proliferation, division, and stem cell population maintenance).

## Related tools

- **dsCellNet** (Upstream pipeline producing the DTW-based L-R interaction expression-change matrix that is clustered) — github.com/songzh523/dsCellNet
- **Mfuzz** (R package providing fuzzy c-means clustering, min.std pre-filtering, and mestimate fuzzifier estimation applied to the L-R expression-change matrix) — github.com/songzh523/dsCellNet
- **R** (Implementation environment; dsCellNet is an open-access R package and Mfuzz clustering is run in R) — github.com/songzh523/dsCellNet

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); developcorrelation(inptim,usedcolo,resloc); # then Mfuzz fuzzy c-means on the L-R expression-change matrix into 8 clusters
```

## Evaluation signals

- Output table partitions the L-R pairs into exactly eight distinct clusters, matching dsCellNet's published clustering
- Every candidate L-R pair is assigned to its maximum-membership cluster (membership values sum to 1 per pair)
- Cluster 1 assignments correspond to interactions peaking at E12.5 in the GSE153164 corticogenesis series
- GO enrichment of the cluster 1 gene list recovers stem cell proliferation, stem cell division, and stem cell population maintenance terms as reported
- Low-variance L-R pairs removed by min.std pre-filtering are absent from the assignment table

## Limitations

- The default L-R gene list size can limit detection of some interactions; users may need to supply supplementary L-R genes
- Number of clusters fixed at eight in dsCellNet's design; other values are not validated by the paper
- Fuzzy memberships are soft, so 'cluster 1 genes' are only the maximum-membership assignments, not exclusive partitions

## Evidence
