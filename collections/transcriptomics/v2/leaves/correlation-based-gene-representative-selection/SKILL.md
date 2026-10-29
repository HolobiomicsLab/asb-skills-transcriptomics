---
name: correlation-based-gene-representative-selection
description: Use when a combined DEAPA/LSAPA/SVAPA gene set must be reduced to representative
  genes and assigned to spatial APA usage patterns across tissue layers of spatially
  barcoded ST data.
when_to_use_negative:
- Input APA data come from imaging-based technologies such as MERFISH/seqFISH, where
  APA sites cannot be identified from spatially barcoded ST data with existing tools
- The goal is spatially variable gene expression rather than spatial APA usage patterns;
  use SPARK/SpatialDE-type expression analyses instead
- Genes have only a single poly(A) site (no APA), since only genes with at least two
  poly(A) sites in the 3' UTR are retained
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing findSpatialPattern for correlation-based selection of
    representative spatial APA genes and findstAPASet to merge DEAPA/LSAPA/SVAPA genes
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Identifies SVAPA genes (adjusted P < 0.05) that feed the combined gene set
    prior to representative selection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:18017168e520eefcbf8d0c0498bb63f1ce1223c482ba28e2ad033060f33db695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - org.Mm.eg.db
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# correlation-based-gene-representative-selection

## Summary

A stAPAminer procedure that selects representative spatial APA genes by computing each gene's Pearson correlation (threshold > 0.5, P < 0.05) against cluster mean APA usage profiles, then groups selected genes by k-means into five spatial patterns corresponding to mouse olfactory bulb (MOB) morphological layers.

## When to use

Use when a combined DEAPA/LSAPA/SVAPA gene set must be reduced to representative genes and assigned to spatial APA usage patterns across tissue layers of spatially barcoded ST data.

## When NOT to use

- Input APA data come from imaging-based technologies such as MERFISH/seqFISH, where APA sites cannot be identified from spatially barcoded ST data with existing tools
- The goal is spatially variable gene expression rather than spatial APA usage patterns; use SPARK/SpatialDE-type expression analyses instead
- Genes have only a single poly(A) site (no APA), since only genes with at least two poly(A) sites in the 3' UTR are retained

## Inputs

- stAPAminer object (stObj) with imputed APA ratio (RUD) matrix and spot coordinates
- Combined non-redundant gene set from DEAPA, LSAPA, and SVAPA genes (findstAPASet output)
- Cluster mean APA usage profiles per morphological layer

## Outputs

- Gene-to-spatial-pattern assignments (e.g., spatial_apa_patterns.csv)
- Five k-means clusters of representative genes corresponding to MOB layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- Per-cluster gene counts via table(patterns$cluster)

## How to apply

First merge DEAPA genes (adjusted P < 0.05 and log2FC > 0.5 from findContrastAPA), LSAPA genes (findAllMarkers, logFC = 0.5), and SVAPA genes (findSVAPA with SPARK adjusted P < 0.05) into a non-redundant set using findstAPASet(stObj). Then run findSpatialPattern(stObj, gene) on the combined gene set and select representative genes whose APA usage profile correlates with the cluster mean profile at Pearson's correlation > 0.5 and P < 0.05. Group the selected genes with k-means into five spatial patterns matching MOB morphological layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL) and summarize per-cluster counts with table(patterns$cluster). Rationale: correlation filtering removes genes whose APA usage variation is not reproducibly tied to a spatial layer mean, yielding the 654 non-redundant 3' UTR APA genes with interpretable spatial patterns that downstream GO enrichment (org.Mm.eg.db, FDR 5%) can be applied to.

## Related tools

- **stAPAminer** (R package providing findSpatialPattern for correlation-based selection of representative spatial APA genes and findstAPASet to merge DEAPA/LSAPA/SVAPA genes) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Identifies SVAPA genes (adjusted P < 0.05) that feed the combined gene set prior to representative selection)

## Examples

```
geneSet <- findstAPASet(stObj); patterns <- findSpatialPattern(stObj, geneSet); table(patterns$cluster)
```

## Evaluation signals

- Every selected gene passes Pearson's correlation > 0.5 and P < 0.05 against a cluster mean APA usage profile
- K-means grouping resolves exactly five spatial patterns corresponding to MOB layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- The combined non-redundant set from findstAPASet yields ~654 3' UTR APA genes with spatial patterns on ST-MOB data
- Per-cluster counts from table(patterns$cluster) are non-empty and each pattern maps to a known morphological layer
- Selected representative genes show reproducible spatial APA patterns across MOB ST replicates

## Limitations

- The k-means pattern assignment depends on prior clustering quality; discrepancies among replicates may reduce reproducibility of spatial patterns and no method mitigates this
- Selection is specific to 3' UTR APA genes; the 601 intronic APA genes with spatial patterns were distinct and may not map to the same five patterns
- Some genes show spatial APA patterns without spatial gene expression patterns (e.g., St8sia5, Myef2, Per1), so expression-based validation alone can miss representatives

## Evidence

- [discussion] selected genes with Pearson's correlation > 0.5 and P < 0.05, as representative genes with spatial APA patterns: [evidence span withheld — anchor status: not_found]
- [discussion] 654 non-redundant genes and five spatial patterns: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data"
- [readme] README workflow with findSpatialPattern and geneSet: "stObj <- findContrastAPA(stObj)
stObj <- findAllMarkers(stObj,logFC = 0.5)
stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
- [intro] objective to identify APA genes with spatial patterns: "APA genes with differential APA usage between morphological layers and genes with global spatial trends in APA usage variation were identified."
