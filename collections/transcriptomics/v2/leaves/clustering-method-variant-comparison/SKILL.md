---
name: clustering-method-variant-comparison
description: Use when scRNA-Seq cells must be assigned to clusters for cell type identification
  and no single clustering method can be trusted a priori; run four SINCERA clustering
  variants on the identical input and pick the scheme supported by permutation tests
  and selective marker expression.
when_to_use_negative:
- The input matrix has not been through SINCERA preprocessing (filtering of low-quality/contaminated
  cells, gene prefiltering, z-score normalization of cell type-selective genes) —
  clustering raw or unfiltered data invalidates the comparison.
- The analysis goal is downstream gene signature prediction or transcriptional regulatory
  network construction on already-defined cell types rather than selecting a clustering
  scheme.
- Only a single fixed clustering scheme is required with no need to compare alternatives
  or validate cluster significance.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: Provides cluster.assignment (hierarchical average/ward linkage, consensus
    clustering, tight clustering), plotMarkers for marker-expression inspection across
    clusters, and cluster.permutation.analysis for cluster significance testing.
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:fff03dc4201f2476c0bda5573794f43b39cf29f0e6d6d7d2abba4123860a0fd5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# clustering-method-variant-comparison

## Summary

Compares multiple clustering schemes (hierarchical average-linkage, hierarchical ward-linkage, consensus clustering, and tight clustering) produced by SINCERA's cluster.assignment function on the same z-scored, cell-type-selective gene matrix, to select a robust cell type identification scheme for single-cell RNA-Seq data.

## When to use

Use when scRNA-Seq cells must be assigned to clusters for cell type identification and no single clustering method can be trusted a priori; run four SINCERA clustering variants on the identical input and pick the scheme supported by permutation tests and selective marker expression.

## When NOT to use

- The input matrix has not been through SINCERA preprocessing (filtering of low-quality/contaminated cells, gene prefiltering, z-score normalization of cell type-selective genes) — clustering raw or unfiltered data invalidates the comparison.
- The analysis goal is downstream gene signature prediction or transcriptional regulatory network construction on already-defined cell types rather than selecting a clustering scheme.
- Only a single fixed clustering scheme is required with no need to compare alternatives or validate cluster significance.

## Inputs

- SINCERA R S4 object containing the preprocessed, z-score transformed expression matrix of cell type-selective genes
- List of known marker genes for plotMarkers inspection

## Outputs

- cluster_assignments.csv: table of cell identifiers and assigned cluster labels for each of the four clustering conditions (hierarchical average-linkage, hierarchical ward-linkage, consensus, tight clustering)
- Per-condition cluster permutation significance results
- Marker-gene expression plots across clusters per clustering condition

## How to apply

Load the preprocessed, z-score transformed expression matrix of cell type-selective genes (produced by cluster.geneSelection and normalization.zscore) into the SINCERA R S4 object. Run the cluster.assignment function with its default settings — hierarchical clustering with average linkage on Pearson's correlation-based distance — then rerun it three additional times, setting the 'clustering.method' parameter to hierarchical clustering with ward linkage, consensus clustering, and tight clustering on the identical input matrix. For each clustering condition, run plotMarkers to inspect expression patterns of known marker genes across clusters: scattered or overlapping marker expression indicates a low-quality clustering scheme. Run the cluster.permutation.analysis function on each scheme to assess cluster significance via cluster membership permutation. Accept a clustering condition only when the permutation analysis supports cluster significance and marker genes show selective, non-overlapping expression; otherwise iterate with different parameter settings.

## Related tools

- **SINCERA** (Provides cluster.assignment (hierarchical average/ward linkage, consensus clustering, tight clustering), plotMarkers for marker-expression inspection across clusters, and cluster.permutation.analysis for cluster significance testing.) — github.com/xu-lab/SINCERA

## Examples

```
Rscript -e 'library(SINCERA); res <- cluster.assignment(x, clustering.method="hierarchical.average"); res.ward <- cluster.assignment(x, clustering.method="hierarchical.ward")'
```

## Evaluation signals

- Cluster membership permutation analysis (cluster.permutation.analysis) supports cluster significance for the chosen clustering condition.
- plotMarkers shows known marker genes with selective (non-overlapping) expression across the defined clusters for the accepted scheme.
- All four clustering conditions were run on the identical z-scored cell type-selective gene matrix, so differences are attributable to the clustering.method parameter only.
- cluster_assignments.csv contains cell identifiers plus cluster labels for all four clustering conditions and row counts match the number of cells in the input S4 object.
- The default condition reproduces SINCERA defaults: hierarchical clustering with average linkage and Pearson's correlation-based distance.

## Limitations

- Scattered or overlapping marker gene expression across clusters signals a low-quality clustering scheme and requires iterating with different parameter settings before accepting any scheme.
- Cell clustering and cell type identification directly influence all downstream analysis in the SINCERA pipeline, so a poor scheme propagates errors to gene signature prediction and driving-force identification.
- Comparison is restricted to the four clustering variants exposed via SINCERA's clustering.method parameter; other scRNA-Seq clusterers (e.g. BackSPIN, SNN-Cliq, RaceID, Seurat) are outside this skill.

## Evidence
