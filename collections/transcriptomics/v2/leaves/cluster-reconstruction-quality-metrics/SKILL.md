---
name: cluster-reconstruction-quality-metrics
description: Use when evaluating which clusters detected by a reference clustering
  (e.g. griph on FACS-purified PBMC setA log10 counts) can be functionally reconstructed
  by a TF/miRNA/kinase-biased autoencoder latent space.
when_to_use_negative:
- When the input is raw fastq or an unfiltered counts table — preprocessing (cellrangerCount,
  filterZeros, counts2log) must precede SCA encoding.
- When the goal is generic clustering quality rather than testing whether a knowledge-biased
  autoencoder latent space (TF/miRNA/kinase/immune signature targets) drives cell
  aggregation.
- 'When miRNA-level validation is expected directly: miRNAs cannot be quantified together
  with mRNAs at single-cell level, so TF-miRNA circuits are not directly measurable.'
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Autoencoder with hidden layers biased by TF-target latent space whose cluster
    reconstruction is scored by QCC/QCM
  repo: github.com/kendomaniac/rCASC
- name: autoencoderAnalysis
  role: rCASC function computing per-cluster QCC and QCM scores from the SCA encoded
    space
  repo: github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework embedding the autoencoder and autoencoderAnalysis modules, run with
    group='docker'
  repo: github.com/kendomaniac/rCASC
- name: Docker
  role: Container runtime required by rCASC docker-based execution of SCA modules
- name: counts2log
  role: rCASC function converting the count table to log10 prior to SCA encoding
  repo: github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:e18e7ffae8cb0478b396659a22726351ae5379882040cfc16c6ad1bf4c8f9d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - autoencoderAnalysis
  - clusterNgriph
  - counts2log
  - rCASC
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# Cluster reconstruction quality metrics (QCC/QCM)

## Summary

Applies the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics from the SCA/rCASC framework to quantify whether a sparsely-connected autoencoder can reconstruct specific single-cell clusters, using a mean QCC and QCM >= 0.5 threshold across multiple runs to declare a cluster reconstructable.

## When to use

Use when evaluating which clusters detected by a reference clustering (e.g. griph on FACS-purified PBMC setA log10 counts) can be functionally reconstructed by a TF/miRNA/kinase-biased autoencoder latent space.

## When NOT to use

- When the input is raw fastq or an unfiltered counts table — preprocessing (cellrangerCount, filterZeros, counts2log) must precede SCA encoding.
- When the goal is generic clustering quality rather than testing whether a knowledge-biased autoencoder latent space (TF/miRNA/kinase/immune signature targets) drives cell aggregation.
- When miRNA-level validation is expected directly: miRNAs cannot be quantified together with mRNAs at single-cell level, so TF-miRNA circuits are not directly measurable.

## Inputs

- setA log10-transformed PBMC count table (produced with counts2log)
- griph-derived reference cluster assignments for the cells (e.g. from clusterNgriph)
- TRRUST TF-target latent space used to bias the SCA hidden layers
- bestPerm permutation output from each SCA run

## Outputs

- sca_qcc_qcm_cluster_metrics.csv with aggregated mean per-cluster QCC and QCM values
- Binary per-cluster reconstruction verdict (mean QCC and QCM >= 0.5 vs < 0.5)

## How to apply

Run the SCA sparsely-connected autoencoder (rCASC autoencoder function, group='docker', scratch.folder set, regularization=TRUE, variational=FALSE, permutation enabled) multiple times with distinct seeds, biasing the hidden layers with an experimentally validated latent space such as TRRUST TF targets on log10-transformed counts (via counts2log). For each run, execute autoencoderAnalysis (group='docker') with nCluster matching the griph reference clustering, the appropriate separator, Sp parameter, and bestPerm from the SCA runs, to compute per-cluster QCC and QCM scores. Aggregate QCC and QCM across the runs into mean per-cluster values and declare a cluster reconstructable only when both mean scores are >= 0.5. This selectivity is the point of the method: SCA reconstructs only the clusters where the encoding space is key to cell aggregation, e.g. on setA only clusters 1 and 2 (B cells and monocytes) pass, as shown in Fig. 2b-c. Write the aggregated per-cluster metrics to a CSV such as sca_qcc_qcm_cluster_metrics.csv for validation.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Autoencoder with hidden layers biased by TF-target latent space whose cluster reconstruction is scored by QCC/QCM) — github.com/kendomaniac/rCASC
- **autoencoderAnalysis** (rCASC function computing per-cluster QCC and QCM scores from the SCA encoded space) — github.com/kendomaniac/rCASC
- **rCASC** (Framework embedding the autoencoder and autoencoderAnalysis modules, run with group='docker') — github.com/kendomaniac/rCASC
- **Docker** (Container runtime required by rCASC docker-based execution of SCA modules)
- **counts2log** (rCASC function converting the count table to log10 prior to SCA encoding) — github.com/kendomaniac/rCASC

## Examples

```
autoencoderAnalysis(group="docker", scratch.folder="/data/scratch", file="log10_setA_counts.csv", nCluster= griph_clusters, separator=",", Sp=sp_value, bestPerm=best_perm)
```

## Evaluation signals

- Only clusters whose mean QCC and mean QCM (aggregated over multiple SCA runs with distinct seeds) are both >= 0.5 are reported as reconstructable.
- On setA with the TRRUST TF-target latent space, the verdict must match Fig. 2b-c: only clusters 1 and 2 (B cells and monocytes) reach the threshold.
- autoencoderAnalysis must be run with nCluster matching the griph reference clustering and the bestPerm from the corresponding SCA run.
- Output file sca_qcc_qcm_cluster_metrics.csv must contain per-cluster mean QCC and QCM columns covering all reference clusters.
- Scores must be aggregated across multiple runs, not from a single SCA run, to reflect encoding robustness.

## Limitations

- SCA reconstructs only specific clusters; clusters below the 0.5 threshold may still be biologically valid but are not supported by the encoding space.
- SCA usage can be challenging for life-science and medical personnel lacking strong computational skills (requires Linux, Docker daemon, rCASC).
- scRNAseq is very noisy and dropout affects lowly expressed genes such as transcription factors and kinases, which the latent space encodes.
- Swarm (Docker Swarm) support for computers-farm execution was still under implementation at publication time.

## Evidence

- [abstract] QCC/QCM definition: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] selectivity rationale: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] setA clusters 1 and 2 validation: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] robustness metrics emphasis: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [methods] Docker requirement: "Docker daemon installed on the machine, for more info see this document"
