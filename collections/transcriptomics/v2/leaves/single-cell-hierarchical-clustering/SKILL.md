---
name: single-cell-hierarchical-clustering
description: Use when a preprocessed, z-score transformed expression matrix of cell
  type-selective genes is available and cells must be assigned to major cell type
  clusters within the SINCERA pipeline.
when_to_use_negative:
- Input is a raw count matrix that has not been filtered for low-quality cells, low-expression
  genes, or z-score normalized — the clustering expects the preprocessed SINCERA matrix
- The analysis goal is downstream of cell type definition, e.g. cell type-specific
  signature gene analysis or driving-force identification — this skill only assigns
  clusters
- Genes lack a defined expression-specificity threshold (no cluster.geneSelection
  / specificity.thresholdSelection output) — clustering on non-selective genes degrades
  cell type separation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SINCERA
  role: Provides the cluster.assignment function (hierarchical, ward, consensus, tight
    clustering), plotMarkers, and cluster.permutation.analysis used to cluster cells
    and identify major cell types
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:4fbf7fd8ebf0b4ca755d8d278d78e3922535e6a05d1588b0883c21262b87116d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-hierarchical-clustering

## Summary

Assigns single cells to initial clusters in the SINCERA pipeline via hierarchical clustering (average linkage with Pearson's correlation distance) on z-scored cell type-selective genes, with consensus, tight, and ward-linkage alternatives and permutation-based cluster significance testing.

## When to use

Use when a preprocessed, z-score transformed expression matrix of cell type-selective genes is available and cells must be assigned to major cell type clusters within the SINCERA pipeline.

## When NOT to use

- Input is a raw count matrix that has not been filtered for low-quality cells, low-expression genes, or z-score normalized — the clustering expects the preprocessed SINCERA matrix
- The analysis goal is downstream of cell type definition, e.g. cell type-specific signature gene analysis or driving-force identification — this skill only assigns clusters
- Genes lack a defined expression-specificity threshold (no cluster.geneSelection / specificity.thresholdSelection output) — clustering on non-selective genes degrades cell type separation

## Inputs

- R S4 SINCERA object holding a z-score transformed expression matrix of cell type-selective genes (output of cluster.geneSelection / specificity.thresholdSelection)
- Known marker genes for inspecting cluster-specific expression

## Outputs

- cluster_assignments.csv containing cell identifiers and assigned cluster labels for each of the four clustering conditions
- Marker-gene expression plots across clusters for each clustering scheme
- Cluster permutation significance statistics per clustering scheme

## How to apply

Load the preprocessed, z-score transformed expression matrix of cell type-selective genes — produced by preprocessing steps filterLowQualityCells, prefilterGenes, and normalization.zscore, with genes selected by cluster.geneSelection — into the SINCERA R S4 object (created via construct). Run cluster.assignment with defaults: hierarchical clustering with average linkage and Pearson's correlation-based distance on the selected z-scored genes. Rerun cluster.assignment with 'clustering.method' set to ward linkage, consensus clustering, and tight clustering on the identical input matrix to compare schemes. For each condition, run plotMarkers to inspect expression of known marker genes across clusters, and run cluster.permutation.analysis to assess cluster significance via cluster membership permutation. Iterate with different parameter settings if marker expression is scattered or overlapping; write cell identifiers and assigned cluster labels for all four conditions to cluster_assignments.csv.

## Related tools

- **SINCERA** (Provides the cluster.assignment function (hierarchical, ward, consensus, tight clustering), plotMarkers, and cluster.permutation.analysis used to cluster cells and identify major cell types) — https://github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA); cl <- cluster.assignment(obj, clustering.method="hierarchical")
```

## Evaluation signals

- cluster.permutation.analysis supports cluster significance for the chosen clustering scheme (cluster membership permutation)
- Known marker genes show selective (non-overlapping) expression across the defined clusters when inspected with plotMarkers
- cluster_assignments.csv contains one row per cell with cluster labels for all four clustering conditions (average-linkage hierarchical, ward, consensus, tight)
- Cell clustering quality is acceptable before proceeding to downstream analysis, since it 'directly influences all downstream analysis'
- Comparing the four clustering conditions shows consistent major cell type structure rather than scattered marker patterns

## Limitations

- Cluster quality is judged indirectly: scattered or overlapping marker expression indicates a low-quality clustering scheme, requiring iteration with different parameter settings
- Cell clustering directly influences all downstream analysis, so a poor clustering scheme propagates errors into gene signature and driving-force analyses
- Relies on precompiled cell type/gene associations (e.g. EBI Expression Atlas) and known marker genes for validation

## Evidence
