---
name: spatially-variable-gene-detection
description: Use when you have an imputed APA usage (RUD) matrix or a gene-spot expression
  count matrix over ST spots and need to identify genes whose APA usage or expression
  varies spatially (adjusted P < 0.05).
when_to_use_negative:
- Your data are imaging-based (MERFISH/seqFISH) without spatially barcoded reads,
  since APA sites can only be identified from spatially barcoded ST data using tools
  like scAPAtrap or Sierra
- Your matrix is a ratio-type RUD matrix that cannot be reasonably transformed into
  a count matrix meeting SPARK's model assumptions
- You seek genes with layer-differential APA usage (use findContrastAPA/findAllMarkers
  for DEAPA/LSAPA, not findSVAPA)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing findSVAPA to run SPARK on the count-transformed imputed
    RUD matrix and findstAPASet to merge DEAPA/LSAPA/SVAPA gene sets
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Spatial variable gene detection engine applied to the transformed APA usage
    matrix (SVAPA) and to the gene expression count matrix (SV)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:bd684ea8bd2232ac20ac6338b5759f2af77b65cb18933b4584cd3f7796c308ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - stAPAminer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatially-variable-gene-detection

## Summary

Detects spatially variable (SV) signal from spatial transcriptomics data using SPARK, adapted by stAPAminer as findSVAPA to test the count-transformed imputed RUD (relative usage of distal poly(A) site) matrix and identify SVAPA genes in the MOB ST data.

## When to use

Use when you have an imputed APA usage (RUD) matrix or a gene-spot expression count matrix over ST spots and need to identify genes whose APA usage or expression varies spatially (adjusted P < 0.05).

## When NOT to use

- Your data are imaging-based (MERFISH/seqFISH) without spatially barcoded reads, since APA sites can only be identified from spatially barcoded ST data using tools like scAPAtrap or Sierra
- Your matrix is a ratio-type RUD matrix that cannot be reasonably transformed into a count matrix meeting SPARK's model assumptions
- You seek genes with layer-differential APA usage (use findContrastAPA/findAllMarkers for DEAPA/LSAPA, not findSVAPA)

## Inputs

- Layer-annotated stAPAminer object containing the KNN-imputed RUD matrix (imputeAPAIndex with k=10)
- Spatial gene-spot expression count matrix (e.g., count.csv) for the parallel SV gene detection
- Spot coordinates (position.txt) defining the spatial layout of spots

## Outputs

- List of SVAPA genes with adjusted P < 0.05 (133 genes in the Replicate 11 example)
- List of SV genes from SPARK on the expression count matrix
- SVAPA–SV intersection counts (16 overlapping genes expected)
- Combined non-redundant set of DEAPA, LSAPA, and SVAPA genes (e.g., spatial_apa_gene_sets.csv)

## How to apply

Prepare the layer-annotated stAPAminer object built from the KNN-imputed RUD matrix (imputeAPAIndex with k=10, borrowing information from the gene-spot count matrix). Because SPARK expects counts, the RUD ratio matrix is transformed into a count matrix to satisfy its input requirement; then call findSVAPA, which applies SPARK to this transformed matrix and calls genes with adjusted P value < 0.05 SVAPA genes. In parallel, apply SPARK to the spatial gene expression count matrix to obtain conventional SV genes; intersecting the two sets reveals their overlap (only 16 genes in ST-MOB Replicate 11), which is expected to be limited since SVAPA and SV genes come from independent APA and expression profiles. Note SVAPA is one of three spatial APA classes stAPAminer identifies (DEAPA, LSAPA, SVAPA), which are merged into a non-redundant set with findstAPASet.

## Related tools

- **stAPAminer** (R package providing findSVAPA to run SPARK on the count-transformed imputed RUD matrix and findstAPASet to merge DEAPA/LSAPA/SVAPA gene sets) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Spatial variable gene detection engine applied to the transformed APA usage matrix (SVAPA) and to the gene expression count matrix (SV))

## Examples

```
stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- findSVAPA returns a non-redundant SVAPA gene set of expected size (~133 genes at adjusted P < 0.05 on Replicate 11)
- DEAPA count check: ~905 total genes, 403 non-redundant, from stObj@DEAPA$all$gene (adjusted P < 0.05, log2FC > 0.5)
- LSAPA per-layer counts via table(stObj@LSAPA$cluster) sum to ~784 genes (findAllMarkers with logFC = 0.5)
- SVAPA–SV overlap should remain small (~16 genes), consistent with the two classes being derived from independent APA and expression profiles
- Imputation diagnostics: clustering of the imputed RUD (makeStCluster, k=30, resolution=0.6) recovers the five MOB layers (GCL, GL, ONL, OPL, MCL)

## Limitations

- The APA ratio matrix must be transformed into a count matrix to meet SPARK's input requirement; such a transformation may not be theoretically reasonable or fully meet the model's assumptions
- No dedicated tool exists for identifying global spatial patterns from ratio-type data, so the count transformation is a workaround
- Missing APA usage entries remaining after imputation iterations are arbitrarily set to 0
- SVAPA and SV gene classes derive from independent profiles, so their overlap is inherently limited (16 genes in Replicate 11)

## Evidence

- [discussion] SVAPA threshold: "Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes"
- [discussion] count transformation for SPARK: "the APA ratio matrix is transformed into a count matrix to meet the input requirement of SPARK; such a transformation may not be theoretically reasonable or fully meet the model’s assumption"
- [discussion] no ratio-type spatial tool: "Currently, there is no tool available for identifying global spatial patterns from ratio-type data, but there are several tools for identifying SV genes"
- [intro] KNN imputation: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [readme] findSVAPA invocation: "stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
- [intro] SV gene detection tools: "including SpatialDE [31], Trendsceek [32], SPARK [33], and SPARK-X [34]"
