---
name: alpha-parameter-sweep-analysis
description: Use when you need to choose a maximum α for ClonoCluster hybrid clustering
  such that cluster number remains comparable to standard transcriptome clustering
  while still showing significant reorganization.
when_to_use_negative:
- When no clone barcode table exists for the cells — α weighting of clone edges is
  meaningless without barcode assignments
- When cell fate is largely extrinsically determined (e.g. cardiomyocyte-directed
  iPSCs or low-dose vemurafenib samples), where clonal information provides limited
  benefit and marker fidelity can decrease at high α
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0085
tools:
- name: ClonoCluster
  role: Performs the hybrid clustering across the α grid by linearly combining transcriptome
    and clone edge weights (α = 0 purely transcriptome, α = 1 purely clone)
  repo: https://github.com/leeprichman/ClonoCluster
- name: irlba
  role: Computes the 25-component PCA input to clonocluster via irlba_wrap(cm, npc
    = 25)
- name: data.table
  role: Loads the count matrix and barcode table with fread
- name: magrittr
  role: Pipe operator used in the count-matrix processing workflow
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:9a4852a77eebfd4d1ee4e9c60bb5cb3c2f42ae4d73485b45b59235bb534df5a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - ROCR
  - data.table
  - irlba
  - magrittr
  - pheatmap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# alpha-parameter-sweep-analysis

## Summary

A procedure from the ClonoCluster paper for sweeping the hybrid-clustering weighting parameter α over a grid (seq(0, 1, by = 0.1)), plotting the number of clusters returned at each α, and using the inflection point to select a sample-specific 'high α' (with 'low α' set at half of it) that keeps hybrid cluster counts comparable to transcriptome-only clustering.

## When to use

Use when you need to choose a maximum α for ClonoCluster hybrid clustering such that cluster number remains comparable to standard transcriptome clustering while still showing significant reorganization.

## When NOT to use

- When no clone barcode table exists for the cells — α weighting of clone edges is meaningless without barcode assignments
- When cell fate is largely extrinsically determined (e.g. cardiomyocyte-directed iPSCs or low-dose vemurafenib samples), where clonal information provides limited benefit and marker fidelity can decrease at high α

## Inputs

- Normalized or scaled count matrix of barcoded cells (gene by cell, e.g. mycountmatrix.tsv, loaded with data.table::fread)
- Table of unique cell IDs and their assigned barcodes (e.g. mybarcodetable.tsv, cell IDs must match the count matrix)

## Outputs

- Cluster assignments for every cell at each α value in seq(0, 1, by = 0.1) (e.g. *_cluster_assignments.txt)
- Table/line graph of cluster number vs α per sample (cluster_size_plots output, Figure 1A / Figure S3)
- Sample-specific 'high α' (0.4–0.7 range) and 'low α' (half of high α) values

## How to apply

Load the normalized/scaled count matrix and barcode table with data.table::fread, convert to a cell-by-gene matrix via dt2m, and compute a 25-component PCA with irlba_wrap(cm, npc = 25). Run clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1), which performs Louvain community detection at each α while weighting transcriptome versus clone edges. Aggregate per-α cluster assignments into a table of cluster counts per α per deposited sample and generate line graphs of alpha vs number of clusters (cluster_size_plots.R, Figures 1A and S3). First identify the resolution value that returns the desired number of clusters from standard transcriptome-only clustering, then set 'high α' just before the inflection point where cluster number rises sharply and 'low α' at half that value. The derived high α values ranged from 0.4 to 0.7 across the nine samples, confirming α selection is sample-specific.

## Related tools

- **ClonoCluster** (Performs the hybrid clustering across the α grid by linearly combining transcriptome and clone edge weights (α = 0 purely transcriptome, α = 1 purely clone)) — https://github.com/leeprichman/ClonoCluster
- **irlba** (Computes the 25-component PCA input to clonocluster via irlba_wrap(cm, npc = 25))
- **data.table** (Loads the count matrix and barcode table with fread)
- **magrittr** (Pipe operator used in the count-matrix processing workflow)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Recovered high-α values fall within the reported 0.4–0.7 range across the samples
- Cluster counts at α = 0 match transcriptome-only clustering at the chosen resolution (res = 1)
- The chosen high α sits just before the inflection point where the cluster-number curve rises sharply
- Low α equals half of the high α value, per the paper's convention
- Number of clusters at the selected high α remains comparable to standard transcriptome cluster counts

## Limitations

- The paper did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so α selection benefits must be evaluated case by case
- The appropriate relative weighting between transcriptome and clone information may differ between endpoint and transitional systems and remains unresolved
- Version numbers for software other than R (>= 3.6) are not stated

## Evidence

- [discussion] first identifying the resolution value that returns the desired number of clusters from standard transcriptome-only clustering: "first identifying the resolution value that returns the desired number of clusters from standard transcriptome-only clustering"
- [discussion] then plotting the number of clusters returned with increasing α: "then plotting the number of clusters returned with increasing α"
- [methods] clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1): "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [intro] We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [readme] generate line graphs of alpha vs number of clusters for all samples seen in figure 1A and Figure S3: [evidence span withheld — anchor status: external_doc]
