---
name: clustering-result-reproducibility-checking
description: Use when a task requires re-running an unsupervised clustering pipeline
  (RCC) on published bulk RNA-seq matrices (e.g., Ivy GAP, GTEx, TCGA SKCM) to confirm
  that reported novel subtypes and cluster counts can be reproduced.
when_to_use_negative:
- The dataset has a known, small number of clusters already established by prior supervised
  annotation — a direct classifier comparison suffices rather than recursive subtype
  re-discovery.
- Input is single-cell data requiring batch integration at scale (millions of cells);
  the article notes integration with scanorama and geometric sketching was only proposed,
  not demonstrated.
- Computational time is severely constrained — the article lists RCC's computational
  time (tied to consensus clustering with k-means) as a limitation.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RCC (RecursiveConsensusClustering)
  role: Core unsupervised recursive consensus clustering algorithm that produces the
    cluster assignments being validated
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used by RCC for dividing the data at each recursion
- name: clusterAttr
  role: Compares RCC clusters against known sample attributes with a minSample filter
    to validate cluster composition
- name: clusterAnnotation
  role: Annotates CT and PAN clusters from TCGA glioblastoma data to confirm the proneural-signature
    subdivision
- name: ssgsea
  role: Quantifies proneural geneSet enrichment in the annotated glioblastoma clusters
- name: geneMarkers
  role: Derives marker genes for the four TCGA SKCM subtypes to check they match reported
    subtype markers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:8ccb93434b9e888f80faf7e40534c264f5fd7dc4fbd3f543e09fb354b5bea4bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - RCC
  - RCC (Recursive Consensus Clustering)
  - RCC (RecursiveConsensusClustering)
  - clusterAnnotation
  - clusterAttr
  - geneMarkers
  - mclust
  - ssgsea
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# clustering-result-reproducibility-checking

## Summary

A validation workflow for reproducing published RCC (Recursive Consensus Clustering) subtype discoveries on bulk transcriptomic datasets (Ivy GAP, GTEx, TCGA pan-cancer) by re-running RCC_clus and its companion functions and checking recovered cluster counts and subtype compositions against the reported structures.

## When to use

Use when a task requires re-running an unsupervised clustering pipeline (RCC) on published bulk RNA-seq matrices (e.g., Ivy GAP, GTEx, TCGA SKCM) to confirm that reported novel subtypes and cluster counts can be reproduced.

## When NOT to use

- The dataset has a known, small number of clusters already established by prior supervised annotation — a direct classifier comparison suffices rather than recursive subtype re-discovery.
- Input is single-cell data requiring batch integration at scale (millions of cells); the article notes integration with scanorama and geometric sketching was only proposed, not demonstrated.
- Computational time is severely constrained — the article lists RCC's computational time (tied to consensus clustering with k-means) as a limitation.

## Inputs

- Bulk gene expression matrix (e.g., Ivy GAP, GTEx human tissue, TCGA pan-cancer expression file)
- Sample info / attribute file mapping samples to known annotations (tissue, cancer type, cluster labels)
- RCC package config_file specifying clustering parameters
- Gene set for signature enrichment (e.g., proneural geneSet)

## Outputs

- Recursive consensus cluster assignments per sample for each dataset
- Cluster-vs-attribute comparison table (clusterAttr output)
- Annotated CT/PAN clusters with proneural signature ssgsea enrichment scores
- Marker gene lists per SKCM subtype (geneMarkers output)
- Survival comparison across clusters

## How to apply

Load the bulk expression matrices together with their sample-info (attribute) files and run RCC_clus with the package config_file to obtain recursive consensus cluster assignments for each dataset (RCC uses ConsensusClusterPlus with k-means as the base algorithm). Compare RCC clusters against known sample attributes using clusterAttr with the attribute column and a minSample filter. For TCGA glioblastoma, apply clusterAnnotation to the CT and PAN clusters to confirm the reported proneural-signature subdivision, then run ssgsea with the proneural geneSet on the annotated clusters to quantify signature enrichment. On GTEx lung tissue, count recovered RCC clusters against the tSNE + k-means baseline (target: eight clusters vs. two). Finally, run geneMarkers on the TCGA SKCM data to derive marker genes for the four metastatic melanoma subtypes and verify they match the reported subtype markers and survival profiles.

## Related tools

- **RCC (RecursiveConsensusClustering)** (Core unsupervised recursive consensus clustering algorithm that produces the cluster assignments being validated) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used by RCC for dividing the data at each recursion)
- **clusterAttr** (Compares RCC clusters against known sample attributes with a minSample filter to validate cluster composition)
- **clusterAnnotation** (Annotates CT and PAN clusters from TCGA glioblastoma data to confirm the proneural-signature subdivision)
- **ssgsea** (Quantifies proneural geneSet enrichment in the annotated glioblastoma clusters)
- **geneMarkers** (Derives marker genes for the four TCGA SKCM subtypes to check they match reported subtype markers)

## Examples

```
RccObj <- RCC_clus(x, config_file); clusterAttr(RccObj, infoFile, attribute, minSample); geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)
```

## Evaluation signals

- TCGA glioblastoma run: annotated CT and PAN clusters reproduce the reported proneural-signature subdivision, confirmed by significant ssgsea enrichment of the proneural geneSet.
- GTEx lung tissue: RCC recovers eight clusters, compared against the two-cluster tSNE + k-means baseline.
- TCGA SKCM: geneMarkers yields four metastatic melanoma subtypes whose markers match the reported subtype markers and show distinct survival profiles.
- Cluster counts and subtype compositions match the reported structures for all three datasets (Ivy GAP, GTEx, TCGA pan-cancer).
- All recovered clusters pass the minSample filter threshold used in clusterAttr and survival calls.

## Limitations

- RCC's computational time is a known limitation, dependent on the underlying base algorithm (consensus clustering with k-means).
- Analysis of large integrated datasets with millions of cells remains undemonstrated; scanorama integration and geometric sketching are only proposed future steps.
- Typical pipelines rely on known annotations and manual cluster identification via visualization; validation here still requires comparing against reported reference structures.

## Evidence

- [abstract] We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets.: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [methods] RCC_clus with config_file: "RCC_clus(x, config_file)"
- [methods] clusterAttr signature: "clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir)"
- [methods] geneMarkers signature: "geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)"
- [discussion] RCC recapitulates complex findings: "using our algorithm, RCC, available as an R package, with just a few commands we can recapitulate findings obtained through multi-step complex analysis"
- [discussion] computational time limitation: "One of the limitations of our algorithm is computational time. This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [readme] README install instructions: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
