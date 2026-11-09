---
name: mammalian-corticogenesis-temporal-biology
description: Use when a developmental-series scRNA-seq time course (e.g., GSE153164
  corticogenesis spanning E12.5–E17.5) must be grouped into temporal expression clusters
  of L-R pairs to identify key time points.
when_to_use_negative:
- Input is a single (stationary) scRNA-seq condition with no time-point series — dsCellNet's
  DTW/fuzzy-clustering layer requires developmental series information to compute
  interaction correlations.
- Goal is a single cell-cell interaction map per condition rather than dynamic temporal
  refinement of key time points.
- The L-R pairs of interest fall outside dsCellNet's default list; the default gene
  list size can miss some interactions (e.g., MG–Neurons), so supplementary L-R genes
  must be supplied first.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: Loads expression/cell-type/time-point inputs (datainput) and provides the
    developmental-series L-R interaction framework whose expression changes are clustered
  repo: https://github.com/songzh523/dsCellNet
- name: Mfuzz
  role: Performs fuzzy c-means clustering of standardized L-R pair expression changes
    into eight clusters with membership values
- name: R
  role: Runtime environment for both dsCellNet (open-access R package, R >= 4.0.3)
    and Mfuzz
  repo: https://github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:3645da51fa1a069429150a76aaa70bfcbb029e6df0e3ef9d3cc6558da33dc677
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mammalian-corticogenesis-temporal-biology

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for clustering developmental-series expression changes of ligand–receptor (L-R) pairs across the GSE153164 corticogenesis time course using dsCellNet and Mfuzz fuzzy c-means, to recover temporal clusters and peak embryonic time points (E12.5, E15.5, E17.5).

## When to use

Use when a developmental-series scRNA-seq time course (e.g., GSE153164 corticogenesis spanning E12.5–E17.5) must be grouped into temporal expression clusters of L-R pairs to identify key time points.

## When NOT to use

- Input is a single (stationary) scRNA-seq condition with no time-point series — dsCellNet's DTW/fuzzy-clustering layer requires developmental series information to compute interaction correlations.
- Goal is a single cell-cell interaction map per condition rather than dynamic temporal refinement of key time points.
- The L-R pairs of interest fall outside dsCellNet's default list; the default gene list size can miss some interactions (e.g., MG–Neurons), so supplementary L-R genes must be supplied first.

## Inputs

- Developmental-series expression matrix of retained L-R pairs (gene x time point, from GSE153164 corticogenesis data)
- Cell-type label file per sample
- Time-point information file (e.g., E12.5, E15.5, E17.5 labels)
- dsCellNet default L-R pair list (5,744 interactions incl. Ramilowski 2015, CellPhoneDB, NATMI, SingleCellSignalR sources)

## Outputs

- mfuzz_cluster_membership.csv with columns gene, cluster id, membership value (membership >= 0.6 core members)
- Eight distinct fuzzy clusters with peak-time annotations (E12.5, E15.5, E17.5 peak clusters)

## How to apply

Load the expression, cell-type, and time-point files of the retained L-R pairs via dsCellNet's datainput function (github.com/songzh523/dsCellNet, R >= 4.0.3). Standardize the L-R pair expression changes across time points to mean 0 and standard deviation 1, as required by Mfuzz. Run Mfuzz fuzzy c-means on the standardized temporal profiles with the default of eight clusters, then filter assignments at membership >= 0.6 to retain core members. Write the gene/cluster/membership table to mfuzz_cluster_membership.csv and verify that the peak clusters match the reported E12.5 (cluster 1, 564 genes), E15.5 (cluster 4, 407 genes), and E17.5 (cluster 5, 306 genes) clusters. Rationale: fuzzy clustering of interactions is dsCellNet's mechanism for refining key time points and connections from developmental series information.

## Related tools

- **dsCellNet** (Loads expression/cell-type/time-point inputs (datainput) and provides the developmental-series L-R interaction framework whose expression changes are clustered) — https://github.com/songzh523/dsCellNet
- **Mfuzz** (Performs fuzzy c-means clustering of standardized L-R pair expression changes into eight clusters with membership values)
- **R** (Runtime environment for both dsCellNet (open-access R package, R >= 4.0.3) and Mfuzz) — https://github.com/songzh523/dsCellNet

## Examples

```
install.packages(paste(yourpath,"dsCellNetv1_1.0.tar.gz",sep=""), type = "source", repos = NULL); out<-datainput(datloc,expnam,typnam,timnam,"FALSE",resloc)
```

## Evaluation signals

- mfuzz_cluster_membership.csv contains eight distinct cluster IDs
- Cluster 1 peaks at E12.5 with 564 core (membership >= 0.6) genes
- Cluster 4 peaks at E15.5 with 407 genes; cluster 5 peaks at E17.5 with 306 genes
- Standardized temporal profiles have mean 0 and standard deviation 1 per L-R pair across time points
- All rows in the membership table have membership >= 0.6 (core-member filter applied)

## Limitations

- dsCellNet's default L-R gene list size can limit detection of some interactions (e.g., MG–Neurons); users must supply supplementary L-R genes to expand coverage.
- The dsCellNet version number is not stated in the article, complicating exact reproduction.
- Cluster count is fixed at the default of eight; alternative numbers of clusters are not explored in the published workflow.

## Evidence
