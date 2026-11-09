---
name: tf-target-network-encoded-representation
description: Use when the goal is to functionally interpret scRNAseq cell clusters
  by encoding them with a TF-target-biased (e.g., TRRUST) autoencoder latent space
  and to identify which clusters the TF network can reconstruct (mean QCC and QCM
  >= 0.5 across runs).
when_to_use_negative:
- The task is generic clustering or normalization of raw scRNAseq counts — SCA requires
  a reference clustering and log10-transformed counts and is meant for functional
  mining, not primary processing.
- You need a well-defined score for a non-TF latent space or expect all clusters to
  be reconstructed — SCA by design reconstructs only clusters where the encoding space
  drives cell aggregation, so absence of QCC/QCM >= 0.5 for other clusters is expected,
  not a failure.
- miRNAs must be quantified alongside mRNAs — it is currently not possible to quantify
  miRNAs together with mRNAs at single-cell level, so TF-miRNA circuit inputs cannot
  be used directly.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: The autoencoder method whose hidden layers are biased by the TRRUST TF-target
    latent space to encode cell clusters
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the autoencoder and autoencoderAnalysis functions used to
    run SCA and compute QCC/QCM
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Container runtime required by rCASC SCA modules (group='docker' invocations,
    containers at docker.io/repbioinfo)
- name: counts2log
  role: rCASC function to convert the raw count table into the log10-transformed counts
    used as SCA input
  repo: https://github.com/kendomaniac/rCASC
- name: autoencoderAnalysis
  role: Computes per-cluster QCC and QCM scores on the SCA-encoded space using bestPerm
    from the SCA runs
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:1bb5006df7cad5b19a26d41dc15b7ce73d1fede625ee80aa93731b6033c4e467
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - autoencoderAnalysis
  - clusterNgriph
  - counts2log
  - rCASC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# tf-target-network-encoded-representation

## Summary

Encodes single-cell RNAseq clusters in a latent space whose hidden layers are biased by experimentally validated transcription-factor target networks (e.g., TRRUST), via a Sparsely-Connected Autoencoder (SCA) implemented in rCASC, and scores reconstruction of clusters with QCC and QCM metrics.

## When to use

Use when the goal is to functionally interpret scRNAseq cell clusters by encoding them with a TF-target-biased (e.g., TRRUST) autoencoder latent space and to identify which clusters the TF network can reconstruct (mean QCC and QCM >= 0.5 across runs).

## When NOT to use

- The task is generic clustering or normalization of raw scRNAseq counts — SCA requires a reference clustering and log10-transformed counts and is meant for functional mining, not primary processing.
- You need a well-defined score for a non-TF latent space or expect all clusters to be reconstructed — SCA by design reconstructs only clusters where the encoding space drives cell aggregation, so absence of QCC/QCM >= 0.5 for other clusters is expected, not a failure.
- miRNAs must be quantified alongside mRNAs — it is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuit inputs cannot be used directly.

## Inputs

- log10-transformed single-cell counts table (e.g., setA FACS-purified PBMC counts via counts2log)
- reference cluster assignments per cell (e.g., griph-derived clusters from clusterNgriph)
- TRRUST TF-target network used to bias the SCA hidden layers

## Outputs

- Encoded (latent) representation per SCA run
- Per-cluster QCC and QCM scores per run, aggregated as mean per-cluster values (e.g., written to sca_qcc_qcm_cluster_metrics.csv)
- Set of reconstructed clusters (mean QCC and QCM >= 0.5, e.g., clusters 1 and 2 = B cells and monocytes of setA)
- Top-ranked transcription factors per reconstructed cluster

## How to apply

Load a log10-transformed counts table (e.g., FACS-purified PBMC setA converted with counts2log) together with reference cluster assignments (e.g., griph-derived clusters). Train the rCASC autoencoder (group='docker', scratch.folder set, regularization=TRUE, variational=FALSE, permutation enabled) with hidden layers biased by the TRRUST TF-target latent space, repeating over multiple runs with distinct seeds to assess robustness. For each run, run autoencoderAnalysis (group='docker', nCluster matching the reference clustering, separator, Sp parameter, bestPerm from the SCA runs) on the encoded space to compute per-cluster QCC (Quality Control of Cluster) and QCM (Quality Control of Model) scores. Aggregate QCC/QCM across runs into mean per-cluster values; only clusters with mean QCC and QCM >= 0.5 are considered reconstructed, indicating that the TF-target encoding space is a key element for those cells' aggregation.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (The autoencoder method whose hidden layers are biased by the TRRUST TF-target latent space to encode cell clusters) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the autoencoder and autoencoderAnalysis functions used to run SCA and compute QCC/QCM) — https://github.com/kendomaniac/rCASC
- **Docker** (Container runtime required by rCASC SCA modules (group='docker' invocations, containers at docker.io/repbioinfo))
- **counts2log** (rCASC function to convert the raw count table into the log10-transformed counts used as SCA input) — https://github.com/kendomaniac/rCASC
- **autoencoderAnalysis** (Computes per-cluster QCC and QCM scores on the SCA-encoded space using bestPerm from the SCA runs) — https://github.com/kendomaniac/rCASC

## Examples

```
R: autoencoder(group="docker", scratch.folder="/data/scratch", file=paste0(folder,"/setA_log10_counts.csv"), cluster=griph_clusters, regularization=TRUE, variational=FALSE, permutation=TRUE, nPerm=100); autoencoderAnalysis(group="docker", scratch.folder="/data/scratch", nCluster=3, separator=",", Sp=1, bestPerm=best_perm)
```

## Evaluation signals

- Only clusters with mean QCC and QCM >= 0.5 across multiple SCA runs are reported as reconstructed; for setA this must be exactly clusters 1 (B cells) and 2 (monocytes), matching Fig. 2b-c.
- Multiple SCA runs with distinct seeds yield consistent per-cluster QCC/QCM means, demonstrating robustness of the encoding.
- autoencoderAnalysis is invoked with nCluster matching the griph reference clustering and bestPerm taken from the SCA permutation runs.
- Top-ranked TFs recovered for reconstructed clusters are known key elements for the corresponding cell types (e.g., B-cell and monocyte TFs for setA clusters 1 and 2).
- The aggregated metrics table (e.g., sca_qcc_qcm_cluster_metrics.csv) contains one mean QCC and mean QCM value per reference cluster.

## Limitations

- SCA reconstructs only specific clusters; clusters without mean QCC/QCM >= 0.5 (e.g., setA cluster 5, naïve T cells) are not failures but indicate the TF-target encoding space is not the key aggregation driver there.
- Transcription factors are lowly expressed genes and can be notably affected by scRNAseq dropout, which may weaken the TF-biased encoding.
- The workflow requires a 64-bit Linux workstation/server with a Docker daemon and can be demanding in setup for users lacking strong computation skills; rCASC modules run in docker.io/repbioinfo containers.

## Evidence

- [abstract] QCC and QCM quantify SCA's reconstruction ability: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [methods] counts2log converts counts to log10: "Converting a count table in log10: **counts2log**"
- [discussion] setA clusters 1 and 2 top TFs are key for B cells and monocytes: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [readme] SCA modules embedded in docker containers for reproducibility: [evidence span withheld — anchor status: not_found]
