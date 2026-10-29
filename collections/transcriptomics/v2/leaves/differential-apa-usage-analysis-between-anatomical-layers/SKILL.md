---
name: differential-apa-usage-analysis-between-anatomical-layers
description: Use when a spatially barcoded ST dataset already has poly(A) site annotations
  and layer labels (e.g., MOB morphological layers), and you need genes with differential
  or spatially patterned APA usage between those layers.
when_to_use_negative:
- Input is raw sequencing reads without poly(A) site annotations — APA sites must
  first be identified with scAPAtrap/Sierra; stAPAminer starts from a PACdataset APA-spot
  matrix
- Data are image-based (MERFISH/seqFISH) — the article notes APA sites can only be
  identified from spatially barcoded ST data with existing scRNA-seq tools
- The question is spatially variable gene expression from a gene count matrix, not
  APA usage ratios — use SPARK/SpatialDE on gene expression instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: R package providing computeAPAIndex, imputeAPAIndex, createStAPAminerObject,
    makeStCluster, findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, drawVolcano,
    and drawSpatialExpress for the differential APA workflow
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Detects spatially variable APA usage (SVAPA genes) after transforming the
    APA ratio matrix to a count matrix; significance at adjusted P < 0.05
- name: Seurat
  role: 'Underlies spot clustering (makeStCluster: resolution, dims 1:10, k=30, 4500
    variable features) that defines the layer labels used as contrast groups'
- name: scAPAtrap
  role: Upstream tool used to identify poly(A) sites from ST reads, producing the
    APA-spot input required before differential testing
- name: movAPA
  role: Provides the PACdataset format in which the poly(A) site matrix (APA.RDA)
    is stored
- name: org.Mm.eg.db
  role: Maps ENTREZID to gene SYMBOL for reporting DEAPA/LSAPA genes in mouse
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:fd04d1cdccf53b5eaf1959b7d21f4f7e4d111c8ba28dd453fd09f835417d02c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - Seurat
  - movAPA
  - org.Mm.eg.db
  - scAPAtrap
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# differential-apa-usage-analysis-between-anatomical-layers

## Summary

Identify DEAPA, LSAPA, and SVAPA genes whose alternative polyadenylation usage differs across spatially resolved anatomical layers (e.g., MOB GCL, GL, ONL, OPL, MCL) using stAPAminer on an imputed RUD (APA usage) matrix.

## When to use

Use when a spatially barcoded ST dataset already has poly(A) site annotations and layer labels (e.g., MOB morphological layers), and you need genes with differential or spatially patterned APA usage between those layers.

## When NOT to use

- Input is raw sequencing reads without poly(A) site annotations — APA sites must first be identified with scAPAtrap/Sierra; stAPAminer starts from a PACdataset APA-spot matrix
- Data are image-based (MERFISH/seqFISH) — the article notes APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools
- The question is spatially variable gene expression from a gene count matrix, not APA usage ratios — use SPARK/SpatialDE on gene expression instead

## Inputs

- Imputed RUD (APA usage) spot-by-poly(A)-site matrix from imputeAPAIndex (RUD_RAW from computeAPAIndex(APA, rownames(position)))
- poly(A) site annotation as a movAPA PACdataset (APA.RDA from stAPAminer extdata)
- gene-spot spatial expression count matrix (count.csv)
- spot barcode coordinate table (position.txt)
- stAPAminer object (stObj) with clusters renamed to morphological layer labels

## Outputs

- DEAPA gene list per layer contrast (adjusted P < 0.05, log2FC > 0.5) from findContrastAPA
- LSAPA gene list from findAllMarkers(stObj, logFC = 0.5)
- SVAPA gene list from findSVAPA (SPARK, adjusted P < 0.05)
- Combined spatial-APA gene set via findstAPASet (e.g., 654 non-redundant 3' UTR APA genes in five major spatial expression patterns)
- Volcano plots (drawVolcano) and per-gene spatial APA usage maps (drawSpatialExpress)

## How to apply

Start from a KNN-imputed RUD matrix built with computeAPAIndex and imputeAPAIndex (k=10, init=TRUE), which borrows information from the spatial gene expression count matrix; if entries remain missing after iteration, set them to 0. Cluster spots with makeStCluster (resolution=0.6–0.7, dims=1:10, k=30, nfeatures=4500), rename clusters to morphological layers via RenameCluster (e.g., 'GCL','GL','ONL','OPL','MCL'), and verify layer separation with computePearsonIndex(stObj,'RUD') plus volcano plots via drawVolcano. Call findContrastAPA for pairwise differential APA usage and findAllMarkers(stObj, logFC=0.5) for layer-significant (LSAPA) genes, applying the threshold adjusted P < 0.05 and log2 fold change > 0.5 for DEAPA calls. Optionally run findSVAPA (SPARK, adjusted P < 0.05) for spatially variable APA usage and merge DEAPA/LSAPA/SVAPA results with findstAPASet into a combined gene set; visualize individual genes with drawSpatialExpress. Retain only 3' UTR APA genes with at least two poly(A) sites in the 3' UTR (and for intronic analysis, sites supported by ≥3 reads in ≥3 spots) before differential testing.

## Related tools

- **stAPAminer** (R package providing computeAPAIndex, imputeAPAIndex, createStAPAminerObject, makeStCluster, findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, drawVolcano, and drawSpatialExpress for the differential APA workflow) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Detects spatially variable APA usage (SVAPA genes) after transforming the APA ratio matrix to a count matrix; significance at adjusted P < 0.05)
- **Seurat** (Underlies spot clustering (makeStCluster: resolution, dims 1:10, k=30, 4500 variable features) that defines the layer labels used as contrast groups)
- **scAPAtrap** (Upstream tool used to identify poly(A) sites from ST reads, producing the APA-spot input required before differential testing)
- **movAPA** (Provides the PACdataset format in which the poly(A) site matrix (APA.RDA) is stored)
- **org.Mm.eg.db** (Maps ENTREZID to gene SYMBOL for reporting DEAPA/LSAPA genes in mouse)

## Examples

```
stObj <- createStAPAminerObject(RUD,position); stObj <- makeStCluster(stObj,resolution=0.6,dims=1:10,k=30,nfeatures=4500); stObj <- findContrastAPA(stObj); stObj <- findAllMarkers(stObj,logFC=0.5); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- DEAPA calls satisfy adjusted P value < 0.05 and log2 fold change > 0.5
- Volcano plots from drawVolcano (e.g., GCL vs GL) show a separable set of significant APA-shifted genes
- Within-layer spot correlations of imputed RUD profiles exceed between-layer correlations (computePearsonIndex source='RUD')
- Combined DEAPA/LSAPA/SVAPA set is non-redundant and reproducible across replicates — e.g., Adarb2 differential APA between GCL and ONL detected in Replicate 11 is also recovered in Replicate 12
- Representative spatial-APA genes show Pearson's correlation > 0.5 with P < 0.05 to their pattern-group mean profile

## Limitations

- Missing APA usage entries remaining after imputation iterations are arbitrarily set to 0
- Transforming the APA ratio matrix into a count matrix for SPARK may not be theoretically reasonable or fully meet the model's assumptions; no dedicated tool exists for global spatial patterns from ratio-type data
- Only 3' UTR APA genes with at least two poly(A) sites are retained; intronic analysis requires sites supported by at least three reads in at least three spots
- Discrepancies among replicates can obscure reproducible spatial patterns, and the article proposes no mitigation method
- Applicable only to spatially barcoded ST data, limiting use with MERFISH/seqFISH

## Evidence

- [discussion] Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes: "Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes"
- [intro] Based on the profile of imputed APA usage, APA genes with differential APA usage between morphological layers and genes with global spatial trends in APA usage variation were identified.: "Based on the profile of imputed APA usage, APA genes with differential APA usage between morphological layers and genes with global spatial trends in APA usage variation were identified."
- [readme] stObj <- findContrastAPA(stObj): "stObj <- findContrastAPA(stObj)
stObj <- findAllMarkers(stObj,logFC = 0.5)
stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
- [readme] stObj <- RenameCluster(stObj,c("GCL","GL","ONL","OPL","MCL")): "stObj <- RenameCluster(stObj,c("GCL","GL","ONL","OPL","MCL"))"
- [methods] RUD <- imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE): "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
