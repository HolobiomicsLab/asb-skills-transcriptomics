---
name: subtype-marker-identification
description: Use when RCC (or another consensus clustering run) has produced cluster
  assignments on an expression matrix and you need per-cluster marker genes and signature-enrichment
  evidence to characterize novel subtypes.
when_to_use_negative:
- When no cluster assignments exist yet — run RCC_clus first; geneMarkers and ssgsea
  operate on clustered output
- When subtypes are already known and only supervised classification is needed, rather
  than novel subtype discovery
- When clusters contain fewer samples than the minSample filter, making attribute
  comparison and survival analysis unreliable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Unsupervised recursive consensus clustering algorithm (R package) that first
    partitions the transcriptomic data into subtypes whose markers are then derived
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used by RCC for dividing the data at each recursion
- name: geneMarkers
  role: Derives marker genes per RCC cluster from the expression matrix (geneMarkers(inputDir,
    exprsFile, sampleOrd, outputDir))
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ssgsea
  role: Quantifies gene-set (e.g., proneural signature) enrichment in annotated clusters
    via ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: clusterAttr
  role: Compares RCC clusters against known sample attributes with a minSample filter
    to confirm subtype compositions
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: clusterAnnotation
  role: Annotates clusters of interest (e.g., CT and PAN clusters in TCGA glioblastoma)
    before signature enrichment
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:c55d8e31e5091a99b27037218d75a180de8d74ce0ec32d0b54d27bb6be269df8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - RCC
  - RCC (Recursive Consensus Clustering)
  - clusterAnnotation
  - clusterAttr
  - geneMarkers
  - mclust
  - ssgsea
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# subtype-marker-identification

## Summary

Derives and validates marker genes for novel subtypes discovered by Recursive Consensus Clustering (RCC) on bulk or single-cell transcriptomic data, using geneMarkers and ssgsea to assign distinct marker signatures and quantify signature enrichment (e.g., proneural geneSet) per cluster.

## When to use

Use when RCC (or another consensus clustering run) has produced cluster assignments on an expression matrix and you need per-cluster marker genes and signature-enrichment evidence to characterize novel subtypes.

## When NOT to use

- When no cluster assignments exist yet — run RCC_clus first; geneMarkers and ssgsea operate on clustered output
- When subtypes are already known and only supervised classification is needed, rather than novel subtype discovery
- When clusters contain fewer samples than the minSample filter, making attribute comparison and survival analysis unreliable

## Inputs

- Gene expression matrix (bulk transcriptomic, e.g., TCGA pan-cancer/SKCM, Ivy GAP, GTEx expression file)
- Sample info / attribute file with cluster assignments from RCC_clus or clusterAnnotation
- Sample attribute column (e.g., cancer type, tissue) for clusterAttr comparison
- Gene set (e.g., proneural signature geneSet) for ssgsea enrichment
- minSample threshold for clusterAttr/survival filtering

## Outputs

- Per-cluster marker gene lists (e.g., markers for the 4 SKCM melanoma subtypes)
- ssGSEA signature-enrichment scores per cluster
- Cluster-vs-attribute comparison tables from clusterAttr
- Cluster annotations (e.g., proneural subdivision of CT/PAN clusters)

## How to apply

After RCC_clus assigns recursive consensus clusters (using the package config_file) to a dataset such as TCGA SKCM, Ivy GAP, or GTEx, run geneMarkers on the expression matrix and cluster/sample order files to derive marker genes for each cluster (e.g., the four metastatic melanoma subtypes in SKCM). Cross-check cluster membership against known sample attributes with clusterAttr, applying the minSample filter to exclude clusters with too few samples. For clusters of interest (e.g., CT and PAN clusters annotated by clusterAnnotation in TCGA glioblastoma), run ssgsea with a predefined geneSet such as the proneural signature to quantify enrichment per cluster. Validate by confirming recovered cluster counts and subtype compositions match the reported structures, and that marker genes match published subtype markers with distinct survival profiles.

## Related tools

- **RCC (Recursive Consensus Clustering)** (Unsupervised recursive consensus clustering algorithm (R package) that first partitions the transcriptomic data into subtypes whose markers are then derived) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used by RCC for dividing the data at each recursion)
- **geneMarkers** (Derives marker genes per RCC cluster from the expression matrix (geneMarkers(inputDir, exprsFile, sampleOrd, outputDir))) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ssgsea** (Quantifies gene-set (e.g., proneural signature) enrichment in annotated clusters via ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)) — https://github.com/MSCTR/RecursiveConsensusClustering
- **clusterAttr** (Compares RCC clusters against known sample attributes with a minSample filter to confirm subtype compositions) — https://github.com/MSCTR/RecursiveConsensusClustering
- **clusterAnnotation** (Annotates clusters of interest (e.g., CT and PAN clusters in TCGA glioblastoma) before signature enrichment) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
RCC_clus(x, config_file); geneMarkers(inputDir, exprsFile, sampleOrd, outputDir); ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)
```

## Evaluation signals

- Recovered cluster counts match reported structures: 4 SKCM subtypes in TCGA melanoma, 8 GTEx lung clusters (vs. 2 by tSNE + k-means baseline), and proneural subdivision of CT/PAN clusters
- Marker genes for each subtype are significantly distinct across clusters and match reported subtype markers
- ssgsea enrichment for the proneural geneSet is significantly higher in the annotated proneural cluster than in other clusters
- Clusters compared via clusterAttr satisfy the minSample filter and correspond to expected sample attribute compositions
- Subtypes show significantly distinct survival profiles when assessed with the survival function

## Limitations

- RCC's computational time is a limitation, dependent on the underlying base algorithm (consensus clustering with k-means)
- Analysis of large integrated datasets with millions of cells is still a next step, not yet demonstrated
- Typical pipelines rely on known annotations and manual cluster identification via visualization, which RCC aims to reduce but marker interpretation may still require domain knowledge

## Evidence

- [abstract] RCC is an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [methods] RCC_clus signature: "RCC_clus(x, config_file)"
- [methods] geneMarkers signature: "geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)"
- [methods] ssgsea signature: "ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)"
- [discussion] RCC identifies additional subtypes and works on bulk and single-cell data: "with just a few commands we can recapitulate findings obtained through multi-step complex analysis and also identify additional subtypes not described in the original studies"
- [readme] README install instructions: "RCC, an unsurpersived clustering algorithm for novel sub type discovery from bulk and single-cell transcriptomic data sets"
