---
name: layer-specific-apa-marker-detection
description: Use when you have a layer-annotated stAPAminer object built from an imputed
  APA usage (RUD) matrix and need to detect DEAPA, LSAPA, and SVAPA gene sets, e.g.
  for ST-MOB replicates or merging into stAPA sets.
when_to_use_negative:
- Input is raw spatially barcoded reads without prior poly(A) site identification
  (scAPAtrap/Sierra) and imputation — the RUD matrix and layer annotations must already
  exist
- 'Data are single-cell scRNA-seq or image-based (MERFISH/seqFISH): APA sites can
  only be identified from spatially barcoded ST data with existing scRNA-seq tools'
- The goal is spatially variable gene detection from the gene expression matrix alone
  — that is plain SPARK SV analysis, not spatial APA usage mining
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: R package providing findContrastAPA (DEAPA), findAllMarkers (LSAPA), findSVAPA,
    and findstAPASet functions applied to the imputed RUD matrix stObj
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Used by findSVAPA (and for SV gene detection) on the count-transformed RUD
    matrix to identify genes with global spatial patterns at adjusted P < 0.05
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:1fb797a6411ca536353d5add1eb745d642fb0d6f8f3a1f5ab13e8261b591dcbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - stAPAminer
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# layer-specific-apa-marker-detection

## Summary

This skill reproduces the stAPAminer workflow that identifies three classes of genes with spatial APA dynamics — DEAPA genes (differential APA usage between MOB morphological layers), LSAPA genes (layer-specific APA markers), and SVAPA genes (global spatial APA usage patterns) — from the kNN-imputed RUD (relative usage degree) matrix of spatially barcoded ST data.

## When to use

Use when you have a layer-annotated stAPAminer object built from an imputed APA usage (RUD) matrix and need to detect DEAPA, LSAPA, and SVAPA gene sets, e.g. for ST-MOB replicates or merging into stAPA sets.

## When NOT to use

- Input is raw spatially barcoded reads without prior poly(A) site identification (scAPAtrap/Sierra) and imputation — the RUD matrix and layer annotations must already exist
- Data are single-cell scRNA-seq or image-based (MERFISH/seqFISH): APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools
- The goal is spatially variable gene detection from the gene expression matrix alone — that is plain SPARK SV analysis, not spatial APA usage mining

## Inputs

- Layer-annotated stAPAminer object (stObj) built from the kNN-imputed RUD matrix (imputeAPAIndex(RUD_RAW, count, k=10))
- Imputed RUD (relative APA usage) matrix from computeAPAIndex/imputeAPAIndex on the poly(A) site PACdataset
- Gene-spot spatial expression count matrix (e.g. count.csv from ST-MOB data) for SV gene comparison via SPARK

## Outputs

- DEAPA gene list with adjusted P values and log2 fold changes (stObj@DEAPA$all$gene)
- LSAPA gene list per morphological layer (stObj@LSAPA$cluster; 784 genes expected)
- SVAPA gene list from SPARK on the count-transformed RUD matrix (133 genes expected)
- Combined non-redundant stAPA gene set from findstAPASet (654 non-redundant 3' UTR APA genes reported)
- SVAPA–SV overlap gene set (16 common genes expected) and CSV export such as spatial_apa_gene_sets.csv

## How to apply

First, run findContrastAPA on the layer-annotated stAPAminer object (layers GCL, GL, ONL, OPL, MCL obtained via makeStCluster, findLabels, RenameCluster) to identify DEAPA genes with adjusted P value < 0.05 and log2 fold change > 0.5, taking unique genes from stObj@DEAPA$all$gene (905 total, 403 non-redundant expected for Replicate 11). Second, run findAllMarkers with logFC = 0.5 on the same object to identify LSAPA genes within each MOB layer, tabulating per-layer counts with table(stObj@LSAPA$cluster) (784 genes expected). Third, run findSVAPA, which applies SPARK to the count-transformed imputed RUD matrix, to detect SVAPA genes at adjusted P < 0.05 (133 genes expected). Optionally intersect SVAPA genes with SV genes obtained by running SPARK on the spatial gene expression count matrix (only 16 common genes expected, since the two classes derive from independent APA and expression profiles), and merge all three lists with findstAPASet into a combined non-redundant stAPA set (654 non-redundant 3' UTR APA genes reported for the original ST-MOB analysis).

## Related tools

- **stAPAminer** (R package providing findContrastAPA (DEAPA), findAllMarkers (LSAPA), findSVAPA, and findstAPASet functions applied to the imputed RUD matrix stObj) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Used by findSVAPA (and for SV gene detection) on the count-transformed RUD matrix to identify genes with global spatial patterns at adjusted P < 0.05)

## Examples

```
stObj <- findContrastAPA(stObj); stObj <- findAllMarkers(stObj,logFC = 0.5); stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- DEAPA call rate matches expected counts: 905 total unique genes in stObj@DEAPA$all$gene, 403 non-redundant for ST-MOB Replicate 11
- table(stObj@LSAPA$cluster) sums to 784 LSAPA genes across the five renamed MOB layers (GCL, GL, ONL, OPL, MCL)
- findSVAPA yields ~133 SVAPA genes at adjusted P < 0.05; SVAPA ∩ SV from SPARK on the expression matrix is ~16 genes
- findstAPASet returns a combined non-redundant set consistent with the reported 654 non-redundant 3' UTR APA genes
- Representative genes (e.g. Pde1c) show coherent spatial APA patterns when plotted with drawSpatialExpress

## Limitations

- The APA ratio matrix must be transformed into a count matrix to meet SPARK's input requirement; such a transformation may not be theoretically reasonable or fully meet the model's assumptions
- SVAPA and SV gene classes are identified from independent APA and gene expression profiles, so their overlap is limited (only 16 common genes)
- Missing APA usage entries remaining after the KNN imputation iteration process are directly set to 0
- APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools such as scAPAtrap or Sierra, limiting applicability to MERFISH/seqFISH
- No dedicated tool exists for identifying global spatial patterns from ratio-type data; count transformation is a workaround

## Evidence

- [discussion] Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes: "Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes"
- [discussion] genes with adjusted P value < 0.05 were considered genes with spatial APA usage patterns: "genes with adjusted P value < 0.05 were considered genes with spatial APA usage patterns"
- [discussion] A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data"
- [discussion] we identified three types of genes with spatial APA dynamics, namely DEAPA, LSAPA, and SVAPA genes: "we identified three types of genes with spatial APA dynamics, namely DEAPA, LSAPA, and SVAPA genes"
- [readme] Particularly, an imputation model based on k-nearest neighbors algorithm was designed for recovering APA signals.: [evidence span withheld — anchor status: external_doc]
