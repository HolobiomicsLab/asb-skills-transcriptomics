---
name: count-transformed-rud-matrix-statistical-testing
description: Use when the imputed APA usage (RUD) matrix from stAPAminer must be tested
  for global spatial variation to identify SVAPA genes in spatially barcoded ST data.
when_to_use_negative:
- The input is raw APA-spot counts or a poly(A) site matrix — the RUD matrix must
  first be computed (computeAPAIndex) and imputed (imputeAPAIndex) before spatial
  testing
- Data are single-cell scRNA-seq without spatial barcodes — spatial pattern testing
  requires spatially resolved spot coordinates
- Data are image-based transcriptomics (MERFISH/seqFISH) — APA sites can only be identified
  from spatially barcoded ST data with existing tools such as scAPAtrap or Sierra
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: Provides the imputed RUD matrix and the findSVAPA wrapper that count-transforms
    the ratio matrix and runs SPARK; findstAPASet merges SVAPA with DEAPA and LSAPA
    lists
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Statistical model applied to the count-transformed RUD matrix to detect spatially
    variable APA (SVAPA) genes at adjusted P < 0.05, and to the gene expression count
    matrix to detect SV genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:49433cddd6ccc744d1033f2ecc02088abb313a864db9b3a24f9d8b201ad46d3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Count-transformed RUD matrix statistical testing

## Summary

Detects spatially variable APA (SVAPA) genes by transforming the imputed RUD (relative usage difference) matrix into a count matrix and applying SPARK to the spatially barcoded ST data, since SPARK requires count input and no dedicated tool exists for ratio-type spatial data.

## When to use

Use when the imputed APA usage (RUD) matrix from stAPAminer must be tested for global spatial variation to identify SVAPA genes in spatially barcoded ST data.

## When NOT to use

- The input is raw APA-spot counts or a poly(A) site matrix — the RUD matrix must first be computed (computeAPAIndex) and imputed (imputeAPAIndex) before spatial testing
- Data are single-cell scRNA-seq without spatial barcodes — spatial pattern testing requires spatially resolved spot coordinates
- Data are image-based transcriptomics (MERFISH/seqFISH) — APA sites can only be identified from spatially barcoded ST data with existing tools such as scAPAtrap or Sierra

## Inputs

- stAPAminer object containing the KNN-imputed RUD matrix (from computeAPAIndex and imputeAPAIndex with k=10) with layer annotations
- Spatial gene expression gene-spot count matrix (e.g., count.csv from the ST-MOB data) for the SV-gene comparison

## Outputs

- List of SVAPA genes with adjusted P values < 0.05 from SPARK on the count-transformed RUD matrix
- SVAPA–SV overlap gene set (intersection of SVAPA and conventional SPARK SV genes)
- Combined non-redundant spatial APA gene set from findstAPASet (DEAPA + LSAPA + SVAPA)

## How to apply

Starting from the stAPAminer object holding the KNN-imputed RUD matrix (imputeAPAIndex with k=10, borrowing information from the spatial gene expression count matrix), call findSVAPA, which transforms the APA ratio matrix into a count matrix to meet SPARK's input requirement and runs SPARK on the transformed data. Retain genes with adjusted P value < 0.05 as SVAPA genes (133 genes in the ST-MOB Replicate 11 example). To assess specificity, apply SPARK independently to the spatial gene expression count matrix to obtain SV genes and intersect with the SVAPA set; a limited overlap (16 genes) is expected because the two classes derive from independent APA and expression profiles. Merge the SVAPA list with DEAPA (adjusted P < 0.05, log2FC > 0.5) and LSAPA (findAllMarkers, logFC = 0.5) sets using findstAPASet to obtain the combined non-redundant spatial APA gene set (654 non-redundant 3' UTR APA genes reported for ST-MOB).

## Related tools

- **stAPAminer** (Provides the imputed RUD matrix and the findSVAPA wrapper that count-transforms the ratio matrix and runs SPARK; findstAPASet merges SVAPA with DEAPA and LSAPA lists) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Statistical model applied to the count-transformed RUD matrix to detect spatially variable APA (SVAPA) genes at adjusted P < 0.05, and to the gene expression count matrix to detect SV genes)

## Examples

```
library(stAPAminer); stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- SVAPA gene count at adjusted P < 0.05 matches the reported benchmark (133 genes for ST-MOB Replicate 11)
- SVAPA–SV intersection is limited (16 common genes expected), reflecting independent APA and expression profiles
- Combined DEAPA + LSAPA + SVAPA set from findstAPASet yields the reported non-redundant total (654 non-redundant 3' UTR APA genes in the article; the task card expects 905 total, 403 non-redundant DEAPA)
- Gene lists and overlap counts export cleanly to spatial_apa_gene_sets.csv
- All post-imputation RUD entries used as SPARK input are numeric counts with no residual missing values (missing entries after iteration are set to 0)

## Limitations

- The APA ratio matrix is transformed into a count matrix to meet SPARK's input requirement; such a transformation may not be theoretically reasonable or fully meet the model's assumptions
- Currently there is no tool available for identifying global spatial patterns from ratio-type data, so count transformation is a workaround
- Missing APA usage entries remaining after the entire imputation iteration process are directly set to 0
- SVAPA and SV gene classes derive from independent APA and expression profiles, limiting their overlap and complicating cross-validation

## Evidence

- [discussion] the APA ratio matrix is transformed into a count matrix to meet the input requirement of SPARK; such a transformation may not be theoretically reasonable or fully meet the model’s assumption: "the APA ratio matrix is transformed into a count matrix to meet the input requirement of SPARK; such a transformation may not be theoretically reasonable or fully meet the model’s assumption"
- [discussion] genes with adjusted P value < 0.05 were considered genes with spatial APA usage patterns: "genes with adjusted P value < 0.05 were considered genes with spatial APA usage patterns"
- [readme] stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj): "stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
- [discussion] Currently, there is no tool available for identifying global spatial patterns from ratio-type data, but there are several tools for identifying SV genes: "Currently, there is no tool available for identifying global spatial patterns from ratio-type data, but there are several tools for identifying SV genes"
- [discussion] A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data"
