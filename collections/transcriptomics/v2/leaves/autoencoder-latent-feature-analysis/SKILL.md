---
name: autoencoder-latent-feature-analysis
description: Use when scRNAseq clusters need functional annotation — e.g., identifying
  cluster-specific transcription factors or minimal gene panels (≤4 genes) from a
  TF-SCA latent-space frequency matrix of a blood-cell dataset.
when_to_use_negative:
- When the goal is de novo gene-network inference from scRNAseq without prior biological
  knowledge — SCA hidden layers require experimentally validated TF/miRNA/kinase target
  sets
- When clusters lack functional interpretability via transcription factors — SCA reconstructs
  only specific clusters, indicating those where the encoding space is key to cell
  aggregation
- When input is raw FASTQ without a counts table — first run indropCounts or cellrangerCount
  before SCA
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Builds the sparse autoencoder latent space from experimentally validated gene-set
    data for functional mining of single-cell clusters
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the SCA module and the cometsc (COMET) function used to
    extract marker panels from the latent-space frequency matrix
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Runs the reproducible containers (docker.io/repbioinfo) embedding the SCA/rCASC
    workflow tools
- name: R / devtools
  role: R environment in which rCASC and its SCA modules are installed and driven
- name: COMET (cometsc)
  role: Ranks gene-feature combinations (≤4 genes, including negated features) per
    cluster by the COMETsc statistic and reports TP/TN rates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:95314ba2d8e005473f35c0dc45f294878c251f306342eb89a3853f902125a9e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - R / devtools
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# autoencoder-latent-feature-analysis

## Summary

Mines single-cell RNAseq clusters for functional features (TFs, miRNAs, kinases) by projecting cells into a Sparsely-Connected Autoencoder (SCA) latent space built from experimentally validated gene-set knowledge, then extracting cluster marker panels with COMET on the latent-space frequency matrix.

## When to use

Use when scRNAseq clusters need functional annotation — e.g., identifying cluster-specific transcription factors or minimal gene panels (≤4 genes) from a TF-SCA latent-space frequency matrix of a blood-cell dataset.

## When NOT to use

- When the goal is de novo gene-network inference from scRNAseq without prior biological knowledge — SCA hidden layers require experimentally validated TF/miRNA/kinase target sets
- When clusters lack functional interpretability via transcription factors — SCA reconstructs only specific clusters, indicating those where the encoding space is key to cell aggregation
- When input is raw FASTQ without a counts table — first run indropCounts or cellrangerCount before SCA

## Inputs

- scRNAseq UMI count table (e.g., from inDrop or 10XGenomics cellrangerCount)
- Clustering output assigning cells to clusters (e.g., setA clusters 1 and 2)
- Experimentally validated gene-set knowledge (TF targets, miRNA targets, kinase targets, immune signatures) for the SCA hidden layers

## Outputs

- SCA latent-space frequency matrix per cluster
- QCC and QCM robustness scores for the SCA encoding
- Ranked cluster-specific marker panels with COMETsc statistic, TP and TN rates (e.g., comet_setA_panels_table1.csv)

## How to apply

First cluster the scRNAseq count table to define cell subpopulations, then run SCA (implemented as an rCASC module inside Docker containers) using hidden layers derived from experimentally validated data such as TF targets, miRNA targets, kinase targets, or cancer-related immune signatures, combining them in a unique SSCA latent space. Assess encoding robustness with the QCM (Quality Control of Model) and QCC (Quality Control of Cluster) metrics, since SCA efficacy comes from reconstructing only clusters where the encoding space drives cell aggregation. From the TF-based SCA latent-space frequency matrix, run the rCASC cometsc function (COMET) separately per cluster, restricting feature combinations to at most 4 genes, and rank combinations by the COMETsc statistic, recording negated features (e.g., KHSRP negation in cluster 2, POU2F2 negation for naïve T cells). Compute TP and TN rates for each top panel and compare against published benchmarks, e.g., cluster 1 (B cells) panel PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997, and cluster 2 (monocytes) panel CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Builds the sparse autoencoder latent space from experimentally validated gene-set data for functional mining of single-cell clusters) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the SCA module and the cometsc (COMET) function used to extract marker panels from the latent-space frequency matrix) — https://github.com/kendomaniac/rCASC
- **Docker** (Runs the reproducible containers (docker.io/repbioinfo) embedding the SCA/rCASC workflow tools)
- **R / devtools** (R environment in which rCASC and its SCA modules are installed and driven)
- **COMET (cometsc)** (Ranks gene-feature combinations (≤4 genes, including negated features) per cluster by the COMETsc statistic and reports TP/TN rates)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Reproduced top COMET panel for setA cluster 1 must be PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997
- Reproduced top COMET panel for setA cluster 2 must be CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46
- QCC and QCM metrics must confirm the SCA model reconstructs the target clusters (valuable aggregation)
- Top-ranked TFs for setA clusters 1 and 2 should correspond to known B-cell and monocyte key elements
- Results written to comet_setA_panels_table1.csv match the Table 1 COMETsc/TP/TN entries

## Limitations

- SCA analysis depends on clustering output: it can only reconstruct specific clusters — those where the encoding space is a key element for cell aggregation
- Transcription factors and kinases are lowly expressed genes notably affected by scRNAseq dropout, biasing the latent features
- miRNAs cannot be quantified together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured
- SCA usage can be challenging for personnel lacking strong computational skills; it requires a 64-bit Linux machine with a Docker daemon (16–32 GB RAM depending on the counting step)

## Evidence

- [abstract] SCA tool and scope: "we report a tool that we developed for the functional mining of single cell clusters based on Sparsely-Connected Autoencoder (SCA)"
- [abstract] SCA latent space from validated knowledge: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [abstract] QCC/QCM metrics: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] setA TF ranking validating B-cell/monocyte features: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] POU2F2 negation refines cluster definition: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [readme] Docker requirement: "Docker daemon installed on the machine, for more info see this document:"
- [readme] SCA as rCASC extension: [evidence span withheld — anchor status: not_found]
- [methods] rCASC installation command: "install_github("kendomaniac/rCASC")"
