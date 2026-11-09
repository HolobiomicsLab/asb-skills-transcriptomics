---
name: single-cell-clustermining-with-autoencoder
description: Use when scRNAseq counts (e.g. setA, 10XGenomics UMI tables) have pre-computed
  cell clusters and the goal is to discover which biological knowledge (TF, miRNA,
  IS, kinase targets) drives clusterization, or to test whether combining knowledge
  groups improves cluster reconstruction.
when_to_use_negative:
- When no pre-existing cluster assignments exist and de novo clustering itself is
  the goal — SCA queries already-defined cell subpopulations rather than performing
  the clustering
- When the aim is to measure miRNA expression directly — the article notes it is not
  possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA
  circuits cannot be directly measured; miRNA effects are inferred via miRNA target-gene
  latent spaces
- When the input is raw FASTQ rather than a counts table — preprocessing (cellrangerCount,
  indropCounts, filterZeros, lorenzFilter) is a separate upstream workflow
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Core autoencoder module that builds knowledge-derived encoded spaces to reconstruct
    single-cell clusters and produce QCM/QCC metrics
  repo: github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: R framework in which SCA is implemented as a module with GUI and Docker-encapsulated
    containers (docker.io/repbioinfo) for reproducible execution
  repo: github.com/kendomaniac/rCASC
- name: Docker
  role: Encapsulates SCA analysis containers (docker.io/repbioinfo) to guarantee computation
    reproducibility
  repo: https://docs.docker.com/engine/installation/
- name: R / devtools
  role: 'Environment and installer for rCASC: install.packages("devtools"); install_github("kendomaniac/rCASC")'
  repo: github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:d6ae8090ae9218adb2eae80134553dbe4b731b6a98bbc9ed69387317e08730a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - R
  - R / devtools
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-clustermining-with-autoencoder

## Summary

Functional mining of single-cell RNAseq clusters using the Sparsely-Connected Autoencoder (SCA/SSCA) within rCASC: encoded latent spaces built from experimentally validated TF, miRNA, kinase targets and immune signatures reconstruct only clusters where the encoding is key to cell aggregation, scored by QCM and QCC metrics.

## When to use

Use when scRNAseq counts (e.g. setA, 10XGenomics UMI tables) have pre-computed cell clusters and the goal is to discover which biological knowledge (TF, miRNA, IS, kinase targets) drives clusterization, or to test whether combining knowledge groups improves cluster reconstruction.

## When NOT to use

- When no pre-existing cluster assignments exist and de novo clustering itself is the goal — SCA queries already-defined cell subpopulations rather than performing the clustering
- When the aim is to measure miRNA expression directly — the article notes it is not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured; miRNA effects are inferred via miRNA target-gene latent spaces
- When the input is raw FASTQ rather than a counts table — preprocessing (cellrangerCount, indropCounts, filterZeros, lorenzFilter) is a separate upstream workflow

## Inputs

- Single-cell UMI counts table with pre-computed cell-cluster assignments (e.g. setA clusters 1-5: B cells, monocytes, HSC, naïve T cell)
- Experimentally validated target/gene-set definitions: TF targets, miRNA targets, kinase targets, cancer-related immune signatures
- Docker daemon on a 64-bit Linux machine; rCASC installed via install_github("kendomaniac/rCASC")

## Outputs

- Per-cluster QCC (Quality Control of Cluster) and QCM (Quality Control of Model) values per latent-space configuration
- Cluster-significance calls (clusters entering the significant reconstruction area, QCM and QCC >= 0.5)
- Cluster-specific functional signatures (e.g. top-ranked TFs per cluster, miR-129-2-3P for HSC)
- Summary table (e.g. table1_ssca_qcm_qcc.csv) mirroring Table 1 / Fig. 5

## How to apply

Load the counts table with cell-cluster assignments into R within the rCASC/Docker environment, then run the SCA module with a single knowledge-group latent space (TF-only, miRNA-only, IS-only, kinase-only) to establish baselines, recording QCM (Quality Control of Model) and QCC (Quality Control of Cluster) per run. Next, run SSCA with combined latent spaces (e.g. TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase), where each hidden layer is derived from experimentally validated targets merged into one encoded space. For each run, compute QCM and QCC and call clusters 'significant' when both metrics reach >=0.5, meaning SCA selectively reconstructed only clusters where the encoding space is a key element of cell aggregation. Compare combined-space results against baselines — on setA, TF+miRNA repositioned cluster 3 (HSC, marker miR-129-2-3P) into the significant area and TF+miRNA+IS relocated clusters 4/5 (naïve T cell), while kinase alone brought no cluster into the significant area. Assemble QCM/QCC values and per-cluster markers into a results table mirroring Table 1 / Fig. 5 of the paper.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Core autoencoder module that builds knowledge-derived encoded spaces to reconstruct single-cell clusters and produce QCM/QCC metrics) — github.com/kendomaniac/SCAtutorial
- **rCASC** (R framework in which SCA is implemented as a module with GUI and Docker-encapsulated containers (docker.io/repbioinfo) for reproducible execution) — github.com/kendomaniac/rCASC
- **Docker** (Encapsulates SCA analysis containers (docker.io/repbioinfo) to guarantee computation reproducibility) — https://docs.docker.com/engine/installation/
- **R / devtools** (Environment and installer for rCASC: install.packages("devtools"); install_github("kendomaniac/rCASC")) — github.com/kendomaniac/rCASC

## Examples

```
library(devtools); install_github("kendomaniac/rCASC"); library(rCASC)
```

## Evaluation signals

- Combined latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) must show higher QCM/QCC than each individual knowledge-group baseline
- Cluster-significance pattern reproduced: cluster 3 (HSC) significant with TF+miRNA; clusters 4/5 significant with TF+miRNA+IS; no cluster significant with kinase alone
- SCA should reconstruct only specific clusters — clusters with QCM and QCC >= 0.5 fall in the significant reconstruction area, others do not
- Per-cluster markers are consistent with biology (e.g. miR-129-2-3P for HSC; POU2F2 negation fitting naïve T cell for cluster 5)
- Output table (table1_ssca_qcm_qcc.csv) contains QCM/QCC for all single and combined latent-space configurations and matches Table 1 / Fig. 5 patterns

## Limitations

- scRNAseq is very noisy and dropout-prone: scRNA-seq produces more zeros than expected, and transcription factors and kinases are lowly expressed genes notably affected by dropout
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured
- Kinase latent space alone brings no cluster into the significant area, so single-knowledge-group runs may yield no significant clusters
- SCA usage can be challenging for life-science personnel lacking strong computation skills, mitigated only partially by the rCASC GUI and Docker containers

## Evidence

- [abstract] QCC and QCM quantify reconstruction quality: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] Combined latent space rationale: "combines different biological knowledge, i.e., TFs, miRNAs, IS, Kinases, in a unique latent space"
- [methods] rCASC/Docker implementation: "Docker daemon installed on the machine, for more info see this document"
- [readme] SCAtutorial README installation: [evidence span withheld — anchor status: not_found]
