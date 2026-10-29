---
name: marker-gene-expression-validation
description: Use when clusters have been assigned by SINCERA cluster.assignment and
  you must confirm predicted cell types are consistent with selectively expressed
  known marker genes before exporting assignments.
when_to_use_negative:
- No prior biological knowledge of cell type markers exists for the tissue — validation
  relies on known markers such as Epcam and Pecam1
- Data have not yet been clustered (no cluster.assignment output) — there are no clusters
  to cross-validate
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SINCERA
  role: Provides plotMarkers, celltype.enrichment (one-tailed Fisher's exact test
    vs. EBI Expression Atlas table), and celltype.validation (rank aggregation) for
    marker-based validation of cluster cell type assignments
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:2bb0db65a1a23fa3497c0ed355858d47d8419c6cfde13fe51367bf4ab50b1be8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# marker-gene-expression-validation

## Summary

Validates SINCERA cell type assignments by checking that known marker genes (e.g., epithelial Epcam, endothelial Pecam1) are selectively expressed in their expected clusters, using plotMarkers and the rank-aggregation-based celltype.validation check.

## When to use

Use when clusters have been assigned by SINCERA cluster.assignment and you must confirm predicted cell types are consistent with selectively expressed known marker genes before exporting assignments.

## When NOT to use

- No prior biological knowledge of cell type markers exists for the tissue — validation relies on known markers such as Epcam and Pecam1
- Data have not yet been clustered (no cluster.assignment output) — there are no clusters to cross-validate

## Inputs

- SINCERA S4 object built with construct() from exprfile (genes x cells FPKM/TPM matrix) and samplefile, after normalization.zscore and cluster.assignment
- Known marker gene list per expected cell type (e.g., Epcam for epithelial, Pecam1 for endothelial)
- Precompiled EBI Expression Atlas-derived cell type and gene association table (built into SINCERA)

## Outputs

- Marker expression plots per cluster (plotMarkers)
- Predicted cell type per cluster from celltype.enrichment (one-tailed Fisher's exact test)
- celltype.validation consistency score between mapped cell types and marker expression
- Cell-to-cluster/cell-type mapping table (e.g., celltype_assignments.csv)

## How to apply

After cluster.assignment, plot known markers per cluster with plotMarkers and check selective expression: Epcam should be restricted to the epithelial cluster and Pecam1 to the endothelial cluster in the E16.5 mouse lung demo. Run celltype.enrichment (one-tailed Fisher's exact test against the precompiled EBI Expression Atlas cell type-gene association table) to assign predicted cell types, then run celltype.validation, which quantitatively assesses consistency between mapped cell types and known marker expression patterns via rank aggregation. Optionally support the check with cluster.diffgenes results (one-tailed Welch's t test, p < 0.05) showing expected markers among cluster-specific signatures, and cluster.permutation.analysis for cluster significance. Accept the assignment only when each predicted cell type matches selectively expressed known markers; otherwise revisit clustering or the specificity threshold.

## Related tools

- **SINCERA** (Provides plotMarkers, celltype.enrichment (one-tailed Fisher's exact test vs. EBI Expression Atlas table), and celltype.validation (rank aggregation) for marker-based validation of cluster cell type assignments) — https://github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA)  # then run plotMarkers, celltype.enrichment, celltype.validation per demo mouselung.e16.5.R
```

## Evaluation signals

- Epcam shows selective expression in the epithelial cluster and Pecam1 in the endothelial cluster (E16.5 mouse lung demo), reproducing the major cell types of Figure 3
- celltype.validation reports consistency between each mapped cell type and the selectively expressed known markers
- Known marker genes appear among cluster.diffgenes outputs (one-tailed Welch's t test, p < 0.05)
- Every cluster receives a predicted cell type from celltype.enrichment before export of celltype_assignments.csv

## Limitations

- Validation is only as good as the known marker set; contaminated or mislabeled cells must have been removed beforehand via filterContaminatedCells using marker coexpression
- celltype.enrichment depends on SINCERA's precompiled EBI Expression Atlas association table, so cell types absent from that table may not be predictable

## Evidence

- [methods] Cross-validate assignments with plotMarkers and celltype.validation: [evidence span withheld — anchor status: not_found]
- [methods] celltype.enrichment uses Fisher's exact test: [evidence span withheld — anchor status: not_found]
