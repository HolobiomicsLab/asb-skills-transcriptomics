---
name: geo-transcriptomics-data-retrieval
description: Use when a analysis requires raw microarray/RNA-seq expression data hosted
  on GEO (a GSE accession) and it must be converted into per-condition gene-by-sample
  tab-separated matrices with a 'Gene' first column before downstream modeling.
when_to_use_negative:
- Expression data is already provided as per-condition tab-separated matrices with
  gene symbols and a 'Gene' first column (e.g. preprocessed TCGA downloads from xenabrowser.net)
- The dataset requires no ID mapping or replicate aggregation, i.e. features are already
  gene-level and sample metadata selection is trivial
- The goal is raw sequenced-read processing rather than matrix-level retrieval — GEOquery
  returns processed expression matrices, not FASTQ files
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: GEOquery
  role: Downloads GEO series data (getGEO with GSEMatrix=TRUE) and exposes sample
    metadata via pData for condition subsetting
- name: biomaRt
  role: Maps platform probe IDs to gene symbols via getBM against the Ensembl BioMart
    (e.g. affy_hugene_1_0_st_v1 to hgnc_symbol)
- name: dplyr
  role: Drops unmapped/empty gene entries and aggregates replicate probes per gene
    with group_by/summarise_all("mean")
- name: R
  role: Environment in which the retrieval, mapping, and export pipeline is scripted
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:00a91870da9c5096a884da93653d1ec2962bc27872f68f643267913fe06346da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GEOquery
  - Python
  - R
  - biomaRt
  - dplyr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# geo-transcriptomics-data-retrieval

## Summary

Retrieve GEO transcriptomics datasets (e.g. GSE27219 B. subtilis stress data, GSE126124 Crohn's disease) using GEOquery, map probe IDs to gene symbols with biomaRt, aggregate replicate probes with dplyr, and export tab-separated disease/control expression matrices ready for differential network tools like BoostDiff.

## When to use

Use when a analysis requires raw microarray/RNA-seq expression data hosted on GEO (a GSE accession) and it must be converted into per-condition gene-by-sample tab-separated matrices with a 'Gene' first column before downstream modeling.

## When NOT to use

- Expression data is already provided as per-condition tab-separated matrices with gene symbols and a 'Gene' first column (e.g. preprocessed TCGA downloads from xenabrowser.net)
- The dataset requires no ID mapping or replicate aggregation, i.e. features are already gene-level and sample metadata selection is trivial
- The goal is raw sequenced-read processing rather than matrix-level retrieval — GEOquery returns processed expression matrices, not FASTQ files

## Inputs

- GEO accession ID (e.g. GSE27219, GSE126124, GSE156063)
- Sample metadata (pData columns such as 'disease type:ch1', 'tissue:ch1') for condition assignment
- biomaRt gene-ID mapping attributes (e.g. 'affy_hugene_1_0_st_v1' to 'hgnc_symbol')

## Outputs

- Tab-separated disease/stress expression matrix with 'Gene' as first column (rows = genes, columns = samples)
- Tab-separated control expression matrix with the same feature set as the disease matrix
- Aggregated gene-level expression table after probe-to-gene mapping and replicate averaging

## How to apply

Call getGEO(<GSE accession>, GSEMatrix=TRUE) via the GEOquery R package and use pData to select the relevant samples by metadata columns (e.g. 'disease type:ch1' == "Control" vs "Crohn's Disease", or tissue == "colon biopsy"). Map platform probe IDs to gene symbols with biomaRt's getBM (e.g. attributes 'affy_hugene_1_0_st_v1' and 'hgnc_symbol' against the Ensembl mart), then use dplyr to drop missing/empty mappings and aggregate replicates per gene with group_by + summarise_all("mean"). Write one tab-separated file per condition (quote=FALSE, sep="\t", row.names=FALSE) with 'Gene' as the first column; the disease and control files must share the identical feature set. For B. subtilis GSE27219, aggregate stress replicates into salt/heat/ethanol matrices versus the smm control matrix; downstream steps (variance filtering, SMOTE oversampling to 48 samples/condition) consume these files directly.

## Related tools

- **GEOquery** (Downloads GEO series data (getGEO with GSEMatrix=TRUE) and exposes sample metadata via pData for condition subsetting)
- **biomaRt** (Maps platform probe IDs to gene symbols via getBM against the Ensembl BioMart (e.g. affy_hugene_1_0_st_v1 to hgnc_symbol))
- **dplyr** (Drops unmapped/empty gene entries and aggregates replicate probes per gene with group_by/summarise_all("mean"))
- **R** (Environment in which the retrieval, mapping, and export pipeline is scripted)

## Examples

```
gset <- getGEO("GSE27219", GSEMatrix=TRUE); df_expr <- data.frame(exprs(gset[[1]])); hgnc_mapping <- getBM(attributes=c("affy_hugene_1_0_st_v1","hgnc_symbol"), filters="affy_hugene_1_0_st_v1", values=rownames(df_expr), mart=mart)
```

## Evaluation signals

- Output files are tab-separated with the literal header column 'Gene' as the first field (README input-format requirement)
- Disease and control matrices contain exactly the same set of features (README: 'the two inputs should have the same set of features')
- Every mapped probe resolves to a non-empty gene symbol; NA/empty-string mappings are removed before aggregation
- Sample counts per condition match the metadata selection (e.g. colon-biopsy Control vs Crohn's Disease subsets for GSE126124)
- Downstream BoostDiff runs accept the files without feature-mismatch errors, confirming rows=genes/columns=samples orientation

## Limitations

- Preprocessing is dataset-dependent: 'Depending on the study and format of the raw data, you might need to map the genes to the desired genes IDs, filter, and/or preprocess them to select specific samples'
- Large GEO series (e.g. GSE126124 Crohn's disease) can exceed local compute, needing an HPC cluster downstream
- Aggregation by mean after probe-to-gene mapping collapses multiple probes per gene, which discards probe-level variance information

## Evidence

- [methods] you can also use the GEOquery package in R: "If you want to analyze other datasets that are available on the GEO database, you can also use the GEOquery package in R."
- [readme] map, filter, preprocess: "Depending on the study and format of the raw data, you might need to map the genes to the desired genes IDs, filter, and/or preprocess them to select specific samples"
- [readme] library(GEOquery); library(biomaRt); library(dplyr): "library(GEOquery)
library(biomaRt)
library(dplyr)"
- [readme] input format: same features, 'Gene' first column: "The rows correspond to genes, while the columns correspond to samples. Note that the two inputs should have the same set of features, where the first column should be named "Gene"."
- [methods] GSE27219 B. subtilis accession: "The *B. subtilis* datasets can be found in GEO with the accession number GSE27219."
