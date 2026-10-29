---
name: mirna-target-functional-annotation
description: Use when scRNA-seq clusters (e.g., setA PBMC clusters) must be functionally
  annotated via miRNA-target regulatory features, using the SCA encoded space to rank
  cluster-specific miRNAs and QC the reconstruction with QCM/QCC.
when_to_use_negative:
- The input already consists of annotated functional signatures or a feature table
  rather than a raw/log-transformed single-cell counts matrix with cluster assignments.
- The goal is de novo cell clustering rather than functional annotation of existing
  clusters — SCA queries subpopulations already defined by clustering analysis.
- The experiment tries to measure miRNAs together with mRNAs at single-cell level
  — the article notes it is currently not possible to quantify miRNAs with mRNAs in
  the same single cell.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Autoencoder whose hidden layer built from miRTarBase miRNA-target gene sets
    produces the encoded space used to rank cluster-specific miRNAs
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions)
    with GUI, guaranteeing reproducible execution
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Containerizes the SCA workflow (docker.io/repbioinfo images) for reproducible
    computation of QCM/QCC and marker rankings
  repo: https://github.com/kendomaniac/docker4seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:431ce1b7de698757fe406108885d62a4571948851ca824acb9f2845768d61d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mirna-target-functional-annotation

## Summary

Functional annotation of single-cell clusters using a Sparsely-Connected Autoencoder (SCA) whose hidden layer is built from experimentally validated miRNA-target gene sets (miRTarBase), implemented as an rCASC module executed in Docker containers and scored with the QCM/QCC metrics.

## When to use

Use when scRNA-seq clusters (e.g., setA PBMC clusters) must be functionally annotated via miRNA-target regulatory features, using the SCA encoded space to rank cluster-specific miRNAs and QC the reconstruction with QCM/QCC.

## When NOT to use

- The input already consists of annotated functional signatures or a feature table rather than a raw/log-transformed single-cell counts matrix with cluster assignments.
- The goal is de novo cell clustering rather than functional annotation of existing clusters — SCA queries subpopulations already defined by clustering analysis.
- The experiment tries to measure miRNAs together with mRNAs at single-cell level — the article notes it is currently not possible to quantify miRNAs with mRNAs in the same single cell.

## Inputs

- Single-cell counts matrix (e.g., setA PBMC-derived UMI counts table)
- Cluster assignment file mapping cells to clusters (clusters 1-5)
- Hidden-layer gene sets of experimentally validated miRNA-target interactions (miRTarBase)
- Optionally, normalized versions of the counts table (log10, CLR, RLE, FQ, SUM, TMM)

## Outputs

- Per-cluster QCM (Quality Control of Model) scores
- Per-cluster QCC (Quality Control of Cluster) scores
- Ranking of cluster-specific miRNA markers derived from the SCA encoded space
- SCA encoded/latent representation of cells per cluster

## How to apply

Load the counts matrix together with pre-defined cluster assignments (e.g., setA clusters 1-5, with cluster 2 = Monocytes and cluster 3 = HSC). Build the SCA hidden layer from miRTarBase experimentally validated miRNA-target gene sets, and run the sparsely-connected autoencoder through the rCASC autoencoder/autoencoderAnalysis functions inside Docker containers (docker.io/repbioinfo), optionally in log10-transformed or global-scaling normalized input (CLR, RLE, TMM, FQ, SUM; TMM/RLE suit UMI data). Compute the per-cluster QCM (Quality Control of Model) and QCC (Quality Control of Cluster) metrics to judge which clusters the SCA encoding actually reconstructs — clusters near the 0.5 threshold (e.g., clusters 2 and 3) are sensitive to normalization choices. Finally, rank cluster-specific miRNA markers from the SCA encoded space and check marker stability across normalizations (e.g., miR-191 and miR-132-3p under TMM; miR-187-3p under FQ and SUM).

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Autoencoder whose hidden layer built from miRTarBase miRNA-target gene sets produces the encoded space used to rank cluster-specific miRNAs) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions) with GUI, guaranteeing reproducible execution) — https://github.com/kendomaniac/rCASC
- **Docker** (Containerizes the SCA workflow (docker.io/repbioinfo images) for reproducible computation of QCM/QCC and marker rankings) — https://github.com/kendomaniac/docker4seq

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCM/QCC values are computed for every cluster in every run (e.g., across all six normalizations) and each cluster's scores are compared to the 0.5 threshold.
- Clusters near the 0.5 QCM/QCC threshold (e.g., setA clusters 2 and 3) shift across normalizations, while low-quality clusters (1, 4, 5) remain minimally affected — reproducing this profile validates the run.
- Top-ranked miRNAs from the encoded space are consistent across normalization schemes and match the expected marker set (miR-191 and miR-132-3p under TMM; miR-187-3p under FQ and SUM).
- The Docker containers (docker.io/repbioinfo) are pulled and used, and results are accompanied by rCASC meta-data preserving functional and computational reproducibility.

## Limitations

- scRNAseq is very noisy, and dropout (more zeros than expected) particularly affects lowly expressed genes such as transcription factors and kinases.
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured; miRNA effects are inferred via target gene sets.
- Normalization of the input counts table changes QCM/QCC mainly for clusters near the 0.5 threshold (e.g., clusters 2 and 3), so conclusions for near-threshold clusters are normalization-dependent.
- SCA usage can be challenging for life-science and medical personnel lacking strong computational skills, requiring Docker and R.

## Evidence

- [abstract] SCA encoded space derived from validated miRNA-target sets grasps cluster features: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [abstract] QCM and QCC quantify SCA reconstruction quality: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] SCA queries subpopulations for functional features driving clusterization: "SCA was exploited to query cell subpopulations to discover the functional features (e.g., TFs, miRNAs, Kinases, etc.) driving cell clusterization."
- [readme] SCA workflow tools embedded in Docker containers for reproducibility: [evidence span withheld — anchor status: not_found]
- [discussion] SCA metrics estimate robustness of the encoding: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
