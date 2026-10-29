---
name: spatial-transcriptomics-data-handling
description: Use when you have a poly(A) site-by-spot matrix plus a gene-spot expression
  matrix and spot coordinates from spatially barcoded ST data, and you need APA usage
  quantification, KNN imputation, spot clustering, or imputation benchmarking against
  scDaPars.
when_to_use_negative:
- Input is raw spatial ST sequencing reads (FASTQ) — first run the PA extraction pipeline
  (umi_tools barcode/UMI extraction, STAR alignment, umi_tools dedup, scAPAtrap) to
  obtain a poly(A) site matrix before using this skill
- Data are MERFISH/seqFISH or other non-barcoded spatial assays — APA sites can only
  be identified from spatially barcoded ST data using existing tools like scAPAtrap
  or Sierra
- The RUD matrix is already imputed and you only need downstream DEAPA/LSAPA/SVAPA
  identification — use the APA-gene-mining skill instead of re-running imputation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: stAPAminer
  role: 'Core R toolkit: computeAPAIndex, imputeAPAIndex, optimalKvalue, createStAPAminerObject,
    makeStCluster, computeIndex, computePearsonIndex, findstAPASet'
  repo: https://github.com/BMILAB/stAPAminer
- name: scDaPars
  role: Baseline imputation method (default nearest-neighbor filling) against which
    stAPAminer's KNN imputation is benchmarked on the same raw RUD matrix
- name: Seurat
  role: Backed by makeStCluster for clustering imputed APA usage of spots (resolution,
    dims, k, nfeatures parameters)
- name: SPARK
  role: Identifies SVAPA genes via findSVAPA on spatially variable APA usage (adjusted
    P < 0.05)
- name: scAPAtrap
  role: Upstream tool to identify poly(A) sites from spatially barcoded ST mapped
    reads, producing the APA-spot input matrix
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:71115ac13921ca0c7900fbe24717aa27b43d579eaa25c4d41a6efd5f6833508a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - ClusterR
  - SPARK
  - Seurat
  - cluster
  - clusterSim
  - fdm2id
  - ggplot2
  - org.Mm.eg.db
  - scAPAtrap
  - scDaPars
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# spatial-transcriptomics-data-handling

## Summary

Handles spatially barcoded ST data for APA analysis with stAPAminer: loading APA-spot count matrices (APA.RDA), gene-spot count matrices (count.csv) and spot coordinates (position.txt), computing raw RUD APA usage, imputing sparse signals with a KNN model, and clustering/imputation-benchmarking on MOB ST data.

## When to use

Use when you have a poly(A) site-by-spot matrix plus a gene-spot expression matrix and spot coordinates from spatially barcoded ST data, and you need APA usage quantification, KNN imputation, spot clustering, or imputation benchmarking against scDaPars.

## When NOT to use

- Input is raw spatial ST sequencing reads (FASTQ) — first run the PA extraction pipeline (umi_tools barcode/UMI extraction, STAR alignment, umi_tools dedup, scAPAtrap) to obtain a poly(A) site matrix before using this skill
- Data are MERFISH/seqFISH or other non-barcoded spatial assays — APA sites can only be identified from spatially barcoded ST data using existing tools like scAPAtrap or Sierra
- The RUD matrix is already imputed and you only need downstream DEAPA/LSAPA/SVAPA identification — use the APA-gene-mining skill instead of re-running imputation

## Inputs

- APA-spot count matrix stored as a movAPA PACdataset (APA.RDA)
- gene-spot count matrix (count.csv)
- spot coordinates table (position.txt)
- raw RUD APA usage matrix (for imputation comparison)

## Outputs

- imputed RUD APA usage matrix (spots x APA sites)
- Seurat-based spot clustering labels mapped to MOB layers (GCL, GL, ONL, OPL, MCL)
- imputation benchmark metrics table (clustering index, within-layer Pearson correlations) per method
- combined non-redundant spatial APA gene set (e.g. 654 3' UTR genes) and gene-to-pattern assignments

## How to apply

Load the movAPA PACdataset APA.RDA, count.csv and position.txt from stAPAminer's extdata, then compute the raw RUD (relative usage difference) matrix with computeAPAIndex(APA, rownames(position)) and impute it via imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE), borrowing information from the spatial gene expression profile. Optionally determine an optimal K with optimalKvalue(RUD_RAW, count, position, ncluster=5, orders=seq(4,14,2)). Build a stAPAminer object from the imputed RUD matrix and coordinates, cluster spots with makeStCluster (Seurat-based; e.g. resolution=0.6-0.7, dims=1:10, k=30, nfeatures=4500), and assess quality with computeIndex and computePearsonIndex(stObj, 'RUD'). For benchmarking, impute the same raw RUD matrix with scDaPars' default nearest-neighbor filling and compare both methods' clustering indices and within-layer Pearson correlations as a grouped boxplot. Downstream, identify DEAPA (adjusted P < 0.05, log2FC > 0.5), LSAPA (findAllMarkers) and SVAPA (findSVAPA, SPARK adjusted P < 0.05) genes, combine them with findstAPASet, and select representative spatial-pattern genes with Pearson correlation > 0.5 and P < 0.05 to the cluster mean profile.

## Related tools

- **stAPAminer** (Core R toolkit: computeAPAIndex, imputeAPAIndex, optimalKvalue, createStAPAminerObject, makeStCluster, computeIndex, computePearsonIndex, findstAPASet) — https://github.com/BMILAB/stAPAminer
- **scDaPars** (Baseline imputation method (default nearest-neighbor filling) against which stAPAminer's KNN imputation is benchmarked on the same raw RUD matrix)
- **Seurat** (Backed by makeStCluster for clustering imputed APA usage of spots (resolution, dims, k, nfeatures parameters))
- **SPARK** (Identifies SVAPA genes via findSVAPA on spatially variable APA usage (adjusted P < 0.05))
- **scAPAtrap** (Upstream tool to identify poly(A) sites from spatially barcoded ST mapped reads, producing the APA-spot input matrix)

## Examples

```
library(stAPAminer); RUD_RAW <- computeAPAIndex(APA, rownames(position)); RUD <- imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE)
```

## Evaluation signals

- Imputed RUD matrix has the same spots as rownames(position) and no remaining NA entries after iterative KNN imputation (residual missing values are set to 0)
- makeStCluster on the imputed RUD recovers the five MOB morphological layers (renamed GCL, GL, ONL, OPL, MCL via RenameCluster)
- stAPAminer imputation should yield higher clustering index (computeIndex) and higher within-layer Pearson correlations (computePearsonIndex, source='RUD') than scDaPars on the same raw matrix (Figure 3E and F)
- Combined DEAPA + LSAPA + SVAPA set is non-redundant; for ST-MOB it totals 654 3' UTR APA genes forming five spatial pattern clusters (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- Representative spatial-pattern genes satisfy Pearson correlation > 0.5 and P < 0.05 against the cluster mean APA usage profile

## Limitations

- Missing APA usage entries that remain after the entire imputation iteration are directly set to 0, and no method determines whether an entry is truly missing before imputation
- The APA ratio matrix must be transformed into a count matrix for SPARK input, which may not be theoretically reasonable or fully meet the model's assumptions; no dedicated tool exists for identifying global spatial patterns from ratio-type data
- Applicability is limited to spatially barcoded ST data (not MERFISH/seqFISH), and methods to mitigate discrepancies among replicate MOB datasets are not yet proposed

## Evidence

- [abstract] an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals: "In particular, an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals"
- [methods] RUD_RAW<-computeAPAIndex(APA,rownames(position)): "RUD_RAW<-computeAPAIndex(APA,rownames(position))"
- [methods] RUD <- imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE): "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [readme] stObj <- makeStCluster(stObj,resolution = 0.6,dims = 1:10,k=30,nfeatures = 4500): [evidence span withheld — anchor status: external_doc]
- [readme] The input of stAPAminer is a poly(A) site matrix with each row being a poly(A) site and each column being a spot.: [evidence span withheld — anchor status: external_doc]
- [discussion] A total of 654 non-redundant 3' UTR APA genes were obtained from the ST-MOB data: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data"
