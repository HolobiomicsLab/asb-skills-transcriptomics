---
name: gene-set-enrichment-analysis-with-comet
description: Use when SCA (QCM/QCC > 0.5) has identified supported clusters and you
  must functionally annotate their cluster-specific genes against immune-signature
  or KEGG gene sets to validate cell identity.
when_to_use_negative:
- Clusters whose QCM or QCC values are <= 0.5, since the SCA encoding space does not
  support those clusters and COMET validation would not be meaningful
- Datasets where cluster-specific genes have not yet been derived from an SCA/rCASC
  clustering analysis
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: COMET (cometsc)
  role: Performs gene-set enrichment (including pathway negations) on cluster-specific
    genes against immune-signature and KEGG gene sets
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the encoded space and cluster-specific genes and the QCM/QCC metrics
    that select which clusters are COMET-validated
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework embedding SCA modules (autoencoder4clustering, autoencoderAnalysis4Clustering)
    executed in Docker containers
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Encapsulates the rCASC/SCA workflow containers (docker.io/repbioinfo) for
    reproducible execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:d70c995c1a68e4ced9ad2aab57d4648c786124fbbe96c92489116cce7ab36782
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-set-enrichment-analysis-with-comet

## Summary

Functional validation of SCA-defined single-cell clusters by running COMET (cometsc) gene-set enrichment on cluster-specific genes against curated gene sets (immune signatures, KEGG), producing p-values that confirm or refute the biological identity of clusters supported by the SCA latent space.

## When to use

Use when SCA (QCM/QCC > 0.5) has identified supported clusters and you must functionally annotate their cluster-specific genes against immune-signature or KEGG gene sets to validate cell identity.

## When NOT to use

- Clusters whose QCM or QCC values are <= 0.5, since the SCA encoding space does not support those clusters and COMET validation would not be meaningful
- Datasets where cluster-specific genes have not yet been derived from an SCA/rCASC clustering analysis

## Inputs

- Cluster-specific gene lists from SCA-supported clusters (e.g. setA clusters 4 and 5)
- QCM/QCC scores from autoencoderAnalysis4Clustering
- Gene-set collections: cancer-related immune signatures and KEGG pathways
- scRNAseq counts matrix with clustering annotation (setA blood-cell dataset)

## Outputs

- COMET enrichment p-values per cluster/gene-set pair (e.g. NK signature 5.75E-54 for cluster 4; ASTHMA KEGG negation 5.35E-84 for cluster 5)
- SSCA signatures (top-ranked genes including negated ones, e.g. POU2F2 negation in cluster 5)
- Combined validation table of QCM/QCC scores and COMET p-values

## How to apply

First complete the SCA analysis (autoencoder4clustering followed by autoencoderAnalysis4Clustering with Sp neighbor-support scoring in rCASC) and confirm QCM and QCC both exceed 0.5 for the target clusters, e.g. cluster 4 (Natural Killer cells) and cluster 5 (Naïve T cells) of the setA blood-cell dataset. Extract the cluster-specific genes for each supported cluster and run COMET (cometsc) against the selected gene-set collections — cancer-related immune signatures and KEGG pathways. Interpret enrichments including negations: for setA, the NK signature was enriched in cluster 4 with p-value 5.75E-54, and the ASTHMA KEGG pathway was enriched as a negation in cluster 5 with p-value 5.35E-84; negation of POU2F2 also refined the naïve T cell identity of cluster 5. Assemble QCM/QCC scores together with the COMET p-values into a single validation table and accept the run only if QCM and QCC are both > 0.5 and the COMET p-values match the reported NK (5.75E-54) and ASTHMA KEGG negation (5.35E-84) results. All steps are executed inside the reproducible Docker containers provided by the rCASC framework to satisfy computation reproducibility.

## Related tools

- **COMET (cometsc)** (Performs gene-set enrichment (including pathway negations) on cluster-specific genes against immune-signature and KEGG gene sets)
- **SCA (Sparsely-Connected Autoencoder)** (Produces the encoded space and cluster-specific genes and the QCM/QCC metrics that select which clusters are COMET-validated) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework embedding SCA modules (autoencoder4clustering, autoencoderAnalysis4Clustering) executed in Docker containers) — https://github.com/kendomaniac/rCASC
- **Docker** (Encapsulates the rCASC/SCA workflow containers (docker.io/repbioinfo) for reproducible execution)

## Examples

```
library(rCASC); autoencoderAnalysis4Clustering(group="docker", scratch.folder="/data/scratch", file=paste0(getwd(),"/setA.csv"), cluster="4,5", Sp=1); cometsc(group="docker", scratch.folder="/data/scratch", file=paste0(getwd(),"/cluster4_genes.csv"), gene.sets="KEGG,immune_signatures")
```

## Evaluation signals

- QCM (Quality Control of Model) and QCC (Quality Control of Cluster) both exceed 0.5 for the majority of cells in the target clusters
- COMET p-value for the NK signature in cluster 4 matches 5.75E-54
- COMET p-value for the ASTHMA KEGG negation in cluster 5 matches 5.35E-84
- Cluster 5 signature contains the POU2F2 negation consistent with a naïve T cell identity
- QCM/QCC scores and COMET p-values are assembled into a single validation table accepted only when all thresholds are met

## Limitations

- SCA reconstructs only specific clusters, so COMET validation is restricted to those clusters where the SCA encoding space is a key element for cell aggregation
- Transcription factors and kinases are lowly expressed and affected by scRNA-seq dropout, which can impact signature genes used for enrichment
- COMET p-values must exactly match the reported reference values for run acceptance; discrepancies invalidate the validation

## Evidence

- [abstract] QCM/QCC and COMET p-value validation: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] immune-signature latent space: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [discussion] cluster 5 POU2F2 negation: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [readme] reproducible Docker execution: [evidence span withheld — anchor status: not_found]
