---
name: differential-apa-usage-analysis
description: Use when you have an imputed 3' UTR RUD (relative usage of distal poly(A)
  site) matrix from spatially barcoded ST data with layer annotations and need to
  classify genes with spatial APA usage patterns.
when_to_use_negative:
- Input is MERFISH/seqFISH or other non-spatially-barcoded data — APA sites can only
  be identified from spatially barcoded ST data with existing tools like scAPAtrap
  or Sierra
- The goal is identifying spatial variation in conventional gene expression rather
  than APA usage variation — use SV gene tools (SPARK, SpatialDE) directly on the
  expression matrix instead
- Only a single poly(A) site exists per gene's 3' UTR — only 3' UTR APA genes with
  at least two poly(A) sites are retained
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing findContrastAPA (DEAPA), findAllMarkers (LSAPA), findSVAPA,
    and findstAPASet functions applied to the imputed RUD matrix
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Spatial pattern detection applied by findSVAPA to the count-transformed imputed
    RUD matrix, and to the gene expression matrix for SV gene comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:67be6aa49a745e0a436df6f162d67fbd1d78676ee6ac1d1d437b73df00f8e30f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-apa-usage-analysis

## Summary

Identify three classes of genes with spatial APA dynamics — DEAPA (layer-to-layer differential APA usage), LSAPA (layer-specific APA), and SVAPA (globally spatially variable APA usage) — from the KNN-imputed RUD matrix using stAPAminer, then merge them with findstAPASet.

## When to use

Use when you have an imputed 3' UTR RUD (relative usage of distal poly(A) site) matrix from spatially barcoded ST data with layer annotations and need to classify genes with spatial APA usage patterns.

## When NOT to use

- Input is MERFISH/seqFISH or other non-spatially-barcoded data — APA sites can only be identified from spatially barcoded ST data with existing tools like scAPAtrap or Sierra
- The goal is identifying spatial variation in conventional gene expression rather than APA usage variation — use SV gene tools (SPARK, SpatialDE) directly on the expression matrix instead
- Only a single poly(A) site exists per gene's 3' UTR — only 3' UTR APA genes with at least two poly(A) sites are retained

## Inputs

- layer-annotated stAPAminer object (stObj) built from the KNN-imputed RUD (imputed APA usage) matrix and spot coordinates
- gene–spot spatial expression count matrix (for SPARK-derived SV gene comparison)
- MOB morphological layer annotations (e.g., GCL, GL, ONL, OPL, MCL)

## Outputs

- stObj@DEAPA$all$gene list of DEAPA genes (e.g., 905 total, 403 non-redundant in ST-MOB Replicate 11)
- stObj@LSAPA$cluster per-layer LSAPA gene assignments (e.g., 784 genes)
- SVAPA gene set at adjusted P < 0.05 (e.g., 133 genes)
- combined non-redundant spatial APA gene set from findstAPASet (e.g., 654 3' UTR APA genes)
- SVAPA–SV overlap counts (e.g., 16 common genes) written to spatial_apa_gene_sets.csv

## How to apply

On a layer-annotated stAPAminer object built from the imputed RUD matrix, run findContrastAPA to detect DEAPA genes between morphological layers (adjusted P < 0.05, log2 fold change > 0.5), then findAllMarkers with logFC = 0.5 to detect LSAPA genes within each MOB layer (tabulated via table(stObj@LSAPA$cluster)). Run findSVAPA, which transforms the RUD matrix into a count matrix and applies SPARK, to detect SVAPA genes at adjusted P < 0.05; verify SVAPA calls by intersecting with conventional SPARK-derived SV genes from the gene–spot expression matrix (overlap expected to be limited, e.g. 16 genes in ST-MOB Replicate 11). Finally merge the DEAPA, LSAPA, and SVAPA lists with findstAPASet into a non-redundant set (654 non-redundant 3' UTR APA genes expected for ST-MOB).

## Related tools

- **stAPAminer** (R package providing findContrastAPA (DEAPA), findAllMarkers (LSAPA), findSVAPA, and findstAPASet functions applied to the imputed RUD matrix) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Spatial pattern detection applied by findSVAPA to the count-transformed imputed RUD matrix, and to the gene expression matrix for SV gene comparison)

## Examples

```
stObj <- findContrastAPA(stObj); stObj <- findAllMarkers(stObj, logFC = 0.5); stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- DEAPA gene counts match expectations: 905 total unique genes from stObj@DEAPA$all$gene with 403 non-redundant genes at adjusted P < 0.05 and log2FC > 0.5
- Per-layer LSAPA counts via table(stObj@LSAPA$cluster) sum to the expected 784 genes
- findSVAPA returns the expected ~133 SVAPA genes at adjusted P < 0.05
- SVAPA–SV intersection is limited (16 common genes), consistent with the two classes being derived from independent APA and expression profiles
- findstAPASet yields 654 non-redundant 3' UTR APA genes; all three class lists and the SVAPA–SV overlap counts are written to spatial_apa_gene_sets.csv

## Limitations

- The APA ratio matrix must be transformed into a count matrix to meet SPARK's input requirement; such a transformation may not be theoretically reasonable or fully meet the model's assumptions
- No dedicated tool exists for identifying global spatial patterns from ratio-type data, so count transformation is a workaround
- SVAPA and SV gene sets are derived from independent APA and expression profiles, so their overlap is expected to be limited
- Missing APA usage entries remaining after imputation iterations are arbitrarily set to 0

## Evidence

- [discussion] DEAPA threshold: "Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes"
- [discussion] SVAPA threshold: "genes with adjusted P value < 0.05 were considered genes with spatial APA usage patterns"
- [discussion] SPARK count transformation workaround: "the APA ratio matrix is transformed into a count matrix to meet the input requirement of SPARK; such a transformation may not be theoretically reasonable or fully meet the model’s assumption"
- [discussion] Three spatial APA gene classes: "we identified three types of genes with spatial APA dynamics, namely DEAPA, LSAPA, and SVAPA genes"
- [readme] README invocation of the three analyses: "stObj <- findContrastAPA(stObj)
stObj <- findAllMarkers(stObj,logFC = 0.5)
stObj <- findSVAPA(stObj)"
