---
name: single-cell-cluster-functional-annotation
description: Use when a clustered scRNAseq counts matrix needs functional annotation
  — e.g. identifying which TFs, miRNAs, kinases, or immune signatures drive cell clusterization,
  or extracting minimal COMET marker gene panels (≤4 genes) for specific clusters.
when_to_use_negative:
- When only raw FASTQ files are available — pre-process with cellrangerCount/indropCounts
  first; this skill annotates clusters from an already-clustered counts matrix
- When the goal is general cluster discovery without functional interpretation — SCA
  is designed to reconstruct only the clusters supported by its encoding space, not
  to cluster all cells de novo
- When trying to measure TF-miRNA circuits directly — miRNAs cannot currently be quantified
  together with mRNAs at single-cell level, so TF-miRNA latent spaces rely on inferred
  target sets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Encodes the scRNAseq data in a latent space derived from experimentally validated
    biological knowledge (TF/miRNA/kinase targets, immune signatures) to mine cluster-specific
    functional features
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the SCA module (autoencoder4clustering, autoencoderAnalysis4Clustering,
    cometsc) with GUI support and reproducible Docker containers
  repo: https://github.com/kendomaniac/rCASC
- name: COMET (cometsc)
  role: Extracts minimal (≤4 gene) cluster-specific marker panels from the SCA latent-space
    frequency matrix and computes COMETsc, TP, and TN statistics
- name: Docker
  role: Required daemon for the rCASC containers that guarantee computational reproducibility
    of the SCA/COMET workflow
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:e2647a73ee3578724eadbf1504e1cc3c204b4cac41b1660847186a3601ea39eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-cluster-functional-annotation

## Summary

Functional mining of single-cell RNAseq clusters using a Sparsely-Connected Autoencoder (SCA) whose latent space is built from experimentally validated data (TF targets, miRNA targets, kinase targets, cancer-related immune signatures), with COMET marker-panel extraction and QCC/QCM quality metrics, implemented as an rCASC module.

## When to use

Use when a clustered scRNAseq counts matrix needs functional annotation — e.g. identifying which TFs, miRNAs, kinases, or immune signatures drive cell clusterization, or extracting minimal COMET marker gene panels (≤4 genes) for specific clusters.

## When NOT to use

- When only raw FASTQ files are available — pre-process with cellrangerCount/indropCounts first; this skill annotates clusters from an already-clustered counts matrix
- When the goal is general cluster discovery without functional interpretation — SCA is designed to reconstruct only the clusters supported by its encoding space, not to cluster all cells de novo
- When trying to measure TF-miRNA circuits directly — miRNAs cannot currently be quantified together with mRNAs at single-cell level, so TF-miRNA latent spaces rely on inferred target sets

## Inputs

- scRNAseq UMI counts matrix (e.g. setA blood-cell dataset) with cluster annotations
- Experimentally validated latent-space definitions: TF targets, miRNA targets, kinase targets, or manually curated cancer immune signatures
- Pre-computed SCA latent-space frequency matrix for COMET marker-panel extraction
- Reference tables of expected COMET panels/statistics (e.g. Table 1 COMETsc, TP, TN values) for validation

## Outputs

- QCM and QCC scores per cluster (both > 0.5 indicates encoding-space support)
- Top COMET marker gene panel per cluster with COMETsc statistic, TP and TN rates (e.g. PAX5, NFAT5, RFXANK, CHD4 for B cells)
- COMET gene-set enrichment p-values (e.g. NK signature 5.75E-54 for cluster 4; ASTHMA KEGG negation 5.35E-84 for cluster 5)
- Validation table (e.g. comet_setA_panels_table1.csv) assembling QCM/QCC scores and COMET p-values

## How to apply

Load the scRNAseq UMI counts matrix together with its cluster annotation and run the SCA workflow inside rCASC (autoencoder4clustering followed by autoencoderAnalysis4Clustering with Sp neighbor-support scoring) within reproducible Docker containers, encoding the hidden layers with the chosen biological knowledge (TF targets, miRNA targets, kinase targets, or a manually curated cancer-immune-signature latent space). Assess whether the SCA encoding space supports each cluster using the QCM (Quality Control of Model) and QCC (Quality Control of Cluster) metrics, accepting clusters where both exceed 0.5 for the majority of cells. Then run the rCASC cometsc function (COMET) on the SCA latent-space frequency matrix per cluster, restricting feature combinations to at most 4 genes and ranking combinations by the COMETsc statistic, recording negated features (e.g. KHSRP negation in cluster 2, POU2F2 negation for naïve T cells). Compute true-positive (TP) and true-negative (TN) rates for each top panel and compare against reference values (e.g. setA cluster 1: PAX5, NFAT5, RFXANK, CHD4, COMETsc 1.45E-49, TP 0.589, TN 0.997; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP, COMETsc 1.19E-46). The rationale is that SCA reconstructs only specific clusters — those where the encoding space is key to cell aggregation — so QCC/QCM above 0.5 indicates functional support before COMET panels are trusted.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Encodes the scRNAseq data in a latent space derived from experimentally validated biological knowledge (TF/miRNA/kinase targets, immune signatures) to mine cluster-specific functional features) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the SCA module (autoencoder4clustering, autoencoderAnalysis4Clustering, cometsc) with GUI support and reproducible Docker containers) — https://github.com/kendomaniac/rCASC
- **COMET (cometsc)** (Extracts minimal (≤4 gene) cluster-specific marker panels from the SCA latent-space frequency matrix and computes COMETsc, TP, and TN statistics)
- **Docker** (Required daemon for the rCASC containers that guarantee computational reproducibility of the SCA/COMET workflow)

## Examples

```
library(devtools); install_github("kendomaniac/rCASC"); autoencoder4clustering(group="docker", scratch.folder="/data/scratch", file=counts_table, clusters=cluster_annotation, latent.space=tf_targets)
```

## Evaluation signals

- QCM and QCC both exceed 0.5 for the majority of cells in each annotated cluster (e.g. setA clusters 4 and 5)
- Reproduced top COMET panels match reference values: cluster 1 COMETsc 1.45E-49, TP 0.589, TN 0.997; cluster 2 COMETsc 1.19E-46
- COMET enrichment p-values reproduce reported results (NK signature 5.75E-54 for cluster 4; ASTHMA KEGG negation 5.35E-84 for cluster 5)
- Each candidate marker panel uses at most 4 genes, including negated features where applicable (KHSRP in cluster 2, POU2F2 in cluster 5)
- Final results table (comet_setA_panels_table1.csv) assembles QCM/QCC scores and COMET p-values without discrepancies against the manuscript values

## Limitations

- scRNAseq is very noisy and dropout produces more zeros than expected, with a greater bias for poorly expressed genes; transcription factors and kinases are lowly expressed and can be notably affected by dropout
- SCA usage could be particularly challenging for life science and medical personnel lacking strong computation skills, mitigated by the rCASC GUI
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured
- Feature combinations in COMET are restricted to at most 4 genes, so larger marker sets are not explored

## Evidence

- [abstract] QCC/QCM threshold for cluster support: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA encoded space origins: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [abstract] Cluster selectivity rationale: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] setA TF validation: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] POU2F2 negation refines naïve T cell cluster: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [readme] Docker-based reproducibility of SCA workflow: [evidence span withheld — anchor status: not_found]
