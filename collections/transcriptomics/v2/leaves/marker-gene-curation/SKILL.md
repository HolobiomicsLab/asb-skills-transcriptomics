---
name: marker-gene-curation
description: Use when bulk RNA-seq expression data must be decomposed into cell-type
  composition trends but no snRNA-seq/scRNA-seq reference profile exists and only
  cell-specific marker genes (gene, cluster, optional fold-change weight) are available.
when_to_use_negative:
- A matched single-cell/snRNA-seq reference profile is available — use reference-based
  BisqueRNA::ReferenceBasedDecomposition instead, which learns gene-specific bulk
  transformations
- Absolute cell-type proportions are needed that can be compared across cell types
  — BisqueMarker only estimates relative abundance differences within a cell type
- The bulk and single-cell data come from different tissues or the single-cell composition
  is known to be severely biased relative to physiology
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: BisqueRNA
  role: Provides MarkerBasedDecomposition, the semi-supervised marker-based decomposition
    that returns relative abundances in bulk.props
  repo: https://github.com/cozygene/bisque
- name: Bisque
  role: Reference implementation of the BisqueMarker model; used to validate that
    marker-based decomposition reproduces reference-based phenotype associations
  repo: https://github.com/cozygene/bisque
- name: Biobase
  role: Wraps the bulk expression matrix as an ExpressionSet, the required input format
    for BisqueRNA decomposition
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:d7fdb296256ddb6e8538b5d09b1886d0a709ed3d8015e934383adc5096e9439b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cozygene/bisque
  tools:
  - Biobase
  - Bisque
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marker-gene-curation

## Summary

Curates a cell-type marker gene set and applies BisqueRNA::MarkerBasedDecomposition (BisqueMarker) to estimate relative cell-type abundances from bulk RNA-seq when no single-cell reference profile is available, then tests phenotype associations (e.g., adipocyte-BMI, T-cell-Matsuda) against reference-based Bisque results.

## When to use

Use when bulk RNA-seq expression data must be decomposed into cell-type composition trends but no snRNA-seq/scRNA-seq reference profile exists and only cell-specific marker genes (gene, cluster, optional fold-change weight) are available.

## When NOT to use

- A matched single-cell/snRNA-seq reference profile is available — use reference-based BisqueRNA::ReferenceBasedDecomposition instead, which learns gene-specific bulk transformations
- Absolute cell-type proportions are needed that can be compared across cell types — BisqueMarker only estimates relative abundance differences within a cell type
- The bulk and single-cell data come from different tissues or the single-cell composition is known to be severely biased relative to physiology

## Inputs

- Bulk RNA-seq expression matrix (genes x samples), convertible to a Biobase ExpressionSet
- Marker gene data frame with columns: gene, cluster (cell type), optional weight (fold-change)

## Outputs

- Relative cell-type abundance estimates in res$bulk.props
- Association statistics (estimates and significance) for cell-type vs. phenotype regressions, e.g. bisque_marker_associations.csv

## How to apply

Load the bulk expression matrix and convert it to a Biobase ExpressionSet with genes as rows and samples as columns. Prepare a marker data frame with gene, cluster (cell type), and optional weight (fold-change) columns, restricting markers to genes overlapping the bulk matrix. Run BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F) and read relative abundances from the returned bulk.props slot. Regress the marker-based relative abundances (e.g., adipocytes, macrophages, T cells) against sample phenotypes (BMI, Matsuda index) and compare the significance pattern to reference-based Bisque results, writing estimates and association statistics to a CSV such as bisque_marker_associations.csv.

## Related tools

- **BisqueRNA** (Provides MarkerBasedDecomposition, the semi-supervised marker-based decomposition that returns relative abundances in bulk.props) — https://github.com/cozygene/bisque
- **Bisque** (Reference implementation of the BisqueMarker model; used to validate that marker-based decomposition reproduces reference-based phenotype associations) — https://github.com/cozygene/bisque
- **Biobase** (Wraps the bulk expression matrix as an ExpressionSet, the required input format for BisqueRNA decomposition)

## Examples

```
library(Biobase); library(BisqueRNA); bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix); res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F); res$bulk.props
```

## Evaluation signals

- res$bulk.props contains relative abundance estimates for every cell type present in the markers data frame
- Qualitative agreement: adipocyte-BMI, macrophage-BMI, and T-cell-Matsuda associations are significant under marker-based mode, matching reference-based Bisque results in Supplementary Table 1
- Marker genes are restricted to those overlapping the bulk matrix genes before decomposition (GetOverlappingGenes)
- Regression of estimated abundances on phenotypes yields per-cell-type association statistics written to bisque_marker_associations.csv

## Limitations

- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types (not proportions)
- It extracts composition trends from normalized bulk expression only; accuracy depends on the quality/specificity of the supplied marker set
- Reference-based Bisque has its own caveats: with few individuals having single-cell data, the reference profile and gene-specific transformations may become unreliable, and estimates reflect the proportions reported by the single-cell technology used

## Evidence

- [intro] semi-supervised marker-based decomposition: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only cell-specific marker genes"
- [methods] methods: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [methods] ExpressionSet format: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [readme] relative abundances caveat: [evidence span withheld — anchor status: external_doc]
- [readme] reference-based alternative: "This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available. Single-cell data is not explicitly required but can be used to identify these marker"
