---
name: marker-based-cell-type-abundance-estimation
description: Use when bulk RNA-seq samples need cell-type proportion/abundance estimates
  but no matched scRNA/snRNA-seq reference exists, and a validated marker gene list
  for the tissue's cell types is available.
when_to_use_negative:
- A matched single-cell/snRNA-seq reference with overlapping individuals is available
  — use Bisque's reference-based decomposition instead, which was the only method
  yielding the significant microglia–Braak association (p = 0.001)
- The goal is absolute cell proportions that can be compared across different cell
  types — marker-based estimates are relative abundances within a cell type and cannot
  be compared between cell types
- No validated cell-type marker genes exist for the tissue
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: BisqueMarker
  role: Semi-supervised decomposition model that extracts trends in cellular composition
    from normalized bulk expression using only cell-specific marker genes
  repo: github.com/cozygene/bisque
- name: BisqueRNA
  role: R package providing MarkerBasedDecomposition() that returns estimates in the
    bulk.props slot of the result list
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet format required for the bulk.eset input
- name: Bisque
  role: Reference-based alternative (ReferenceBasedDecomposition) used for comparison
    against marker-based estimates in phenotype association analyses
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:27963fbd78500283ddef6ffa4eb298573615346a01893b4d89f4c92bb69f4739
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# marker-based-cell-type-abundance-estimation

## Summary

Estimate cell-type abundance in bulk RNA-seq with BisqueRNA::MarkerBasedDecomposition when a matched single-cell reference profile is unavailable, using only a set of known cell-type-specific marker genes to extract relative abundance trends across individuals.

## When to use

Use when bulk RNA-seq samples need cell-type proportion/abundance estimates but no matched scRNA/snRNA-seq reference exists, and a validated marker gene list for the tissue's cell types is available.

## When NOT to use

- A matched single-cell/snRNA-seq reference with overlapping individuals is available — use Bisque's reference-based decomposition instead, which was the only method yielding the significant microglia–Braak association (p = 0.001)
- The goal is absolute cell proportions that can be compared across different cell types — marker-based estimates are relative abundances within a cell type and cannot be compared between cell types
- No validated cell-type marker genes exist for the tissue

## Inputs

- Bulk RNA-seq expression count matrix (e.g., 628 DLPFC bulk samples) convertible to a Biobase ExpressionSet
- Named list of known cell-type-specific marker genes
- Optional scRNA-seq/snRNA-seq data to define the marker gene list
- Sample phenotypes (e.g., Braak stage, sex, age) for downstream association testing

## Outputs

- Estimated relative cell-type abundances per bulk sample (res$bulk.props)
- Per-cell-type association statistics when proportions are regressed on phenotypes (e.g., linear regression on Braak stage with age and sex covariates)

## How to apply

Convert the bulk expression counts into a Biobase ExpressionSet (bulk.eset) and prepare a named list of cell-type marker genes. Run BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F) and extract estimates from the returned list's bulk.props slot. Unlike Bisque's reference-based mode, this semi-supervised model estimates relative differences in abundance of a cell type across individuals from normalized bulk expression, so it does not learn gene-specific bulk transformations or yield absolute proportions. In the DLPFC replication, marker-based (BisqueMarker) estimates did not reproduce the cross-cell-type association with Braak stage that reference-based Bisque gave (microglia p = 0.001), so downstream phenotype regressions should be interpreted per cell type rather than compared across cell types. Optionally supply single-cell data only to identify the marker genes feeding the model.

## Related tools

- **BisqueMarker** (Semi-supervised decomposition model that extracts trends in cellular composition from normalized bulk expression using only cell-specific marker genes) — github.com/cozygene/bisque
- **BisqueRNA** (R package providing MarkerBasedDecomposition() that returns estimates in the bulk.props slot of the result list) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet format required for the bulk.eset input)
- **Bisque** (Reference-based alternative (ReferenceBasedDecomposition) used for comparison against marker-based estimates in phenotype association analyses) — github.com/cozygene/bisque

## Examples

```
res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F); props <- res$bulk.props
```

## Evaluation signals

- MarkerBasedDecomposition returns a list whose bulk.props element contains estimated abundances for every bulk sample and cell type
- Estimates should capture relative abundance differences within a cell type across individuals, not absolute proportions summing to 1
- In the DLPFC replication, marker-based (BisqueMarker) estimates should NOT yield the significant positive microglia–Braak association (p = 0.001) that reference-based Bisque produces
- Internal filter utilities (FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingGenes) run cleanly and the marker list is restricted to genes overlapping the bulk matrix

## Limitations

- Marker-based estimates are relative differences in abundances that cannot be compared across cell types
- Captures relative abundances of a cell type across individuals; these are not proportions
- Without a single-cell reference, no gene-specific bulk transformations are learned, so accuracy is lower than reference-based Bisque when technical variation between single-cell and bulk technologies exists
- Single-cell data can only be used indirectly to identify marker genes

## Evidence

- [intro] BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only"
- [discussion] BisqueMarker provides cell-type abundance estimations using only known marker genes.: "BisqueMarker provides cell-type abundance estimations using only known marker genes."
- [discussion] it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types.: "it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types."
- [readme] This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available.: "This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available."
- [methods] res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F): "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
