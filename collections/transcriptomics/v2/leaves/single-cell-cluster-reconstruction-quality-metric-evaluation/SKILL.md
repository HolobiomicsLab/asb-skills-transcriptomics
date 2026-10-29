---
name: single-cell-cluster-reconstruction-quality-metric-evaluation
description: Use when an scRNAseq or spatial transcriptomics counts table has been
  clustered (e.g., 9 SIMLR clusters on a Visium breast cancer section) and you need
  to determine which clusters the SCA autoencoder reconstructs for the majority of
  cells before downstream functional annotation.
when_to_use_negative:
- Clusters were not defined beforehand — SCA requires pre-clustered cells; use a clustering
  tool (e.g., SIMLR in rCASC) first
- You only need bulk or pseudo-bulk autoencoding where per-cluster reconstruction
  quality is not meaningful
- Input is already a TF activity or feature table rather than a raw counts matrix
  accepted by SCA
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: rCASC
  role: Framework hosting the SCA autoencoder module whose QCC/QCM outputs are evaluated;
    install_github("kendomaniac/rCASC")
  repo: https://github.com/kendomaniac/rCASC
- name: SCA (Sparsely-Connected Autoencoder)
  role: Generates per-cluster QCC (cluster reconstruction) and QCM (model quality)
    metrics from TF-target-informed latent spaces
  repo: https://github.com/kendomaniac/rCASC
- name: SIMLR
  role: Provides the cluster labels (9 clusters on the Visium breast cancer section)
    that are then quality-checked with QCC/QCM
- name: Docker
  role: Runs the rCASC/SCA containers (docker.io/repbioinfo) ensuring reproducible
    computation of the metrics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:7ba46cddffb4af7541073b8b078d4a473cf760fe09098ce1ff4fadd26ff70d85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - SIMLR
  - rCASC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-cluster-reconstruction-quality-metric-evaluation

## Summary

Evaluates which single-cell/spatial-transcriptomics clusters are meaningfully reconstructed by the rCASC Sparsely-Connected Autoencoder (SCA) using the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics with a >0.5 threshold per cell. It decides which clusters are worth downstream TF-ranking (e.g., COMET) rather than blindly analyzing all clusters.

## When to use

Use when an scRNAseq or spatial transcriptomics counts table has been clustered (e.g., 9 SIMLR clusters on a Visium breast cancer section) and you need to determine which clusters the SCA autoencoder reconstructs for the majority of cells before downstream functional annotation.

## When NOT to use

- Clusters were not defined beforehand — SCA requires pre-clustered cells; use a clustering tool (e.g., SIMLR in rCASC) first
- You only need bulk or pseudo-bulk autoencoding where per-cluster reconstruction quality is not meaningful
- Input is already a TF activity or feature table rather than a raw counts matrix accepted by SCA

## Inputs

- Raw counts table for the Visium breast cancer Block A Section 1 (or other scRNAseq/spatial counts matrix)
- Cluster annotation assigning spots/cells to clusters (e.g., SIMLR labels for 9 clusters)
- Trained SCA autoencoder models per cluster (TF-target-informed sparsely-connected hidden layers)

## Outputs

- Per-cluster QCC values (fraction of cells where the cluster is reconstructed)
- Per-cluster QCM values (neural network model quality)
- List of clusters passing QCC > 0.5 and QCM > 0.5 for the majority of cells (e.g., only cluster 7)

## How to apply

For each pre-defined cluster (e.g., the 9 SIMLR clusters from the Visium breast cancer Block A Section 1), train the rCASC SCA autoencoder whose hidden layers are sparsely connected according to experimentally validated TF-target (and optionally miRNA, kinase, immune-signature) links, using the raw counts as input. After training, compute per-cluster QCC and QCM: QCC quantifies the ability of SCA to reconstruct a valuable cell cluster and QCM evaluates the neural network model quality. Apply the >0.5 threshold per metric and accept a cluster only if QCC and QCM exceed 0.5 for the majority of its cells; in the Visium breast cancer case only cluster 7 passed, so downstream TF ranking (COMET on the SCA latent space) was restricted to that cluster. This gating rationale stems from SCA's property of reconstructing only specific clusters, i.e., those where the SCA encoding space is a key element for cell aggregation.

## Related tools

- **rCASC** (Framework hosting the SCA autoencoder module whose QCC/QCM outputs are evaluated; install_github("kendomaniac/rCASC")) — https://github.com/kendomaniac/rCASC
- **SCA (Sparsely-Connected Autoencoder)** (Generates per-cluster QCC (cluster reconstruction) and QCM (model quality) metrics from TF-target-informed latent spaces) — https://github.com/kendomaniac/rCASC
- **SIMLR** (Provides the cluster labels (9 clusters on the Visium breast cancer section) that are then quality-checked with QCC/QCM)
- **Docker** (Runs the rCASC/SCA containers (docker.io/repbioinfo) ensuring reproducible computation of the metrics)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC"); # train SCA per cluster then inspect QCC/QCM output tables with threshold > 0.5
```

## Evaluation signals

- QCC > 0.5 and QCM > 0.5 for the majority of cells in exactly the accepted cluster(s); in the reference case only cluster 7 passes out of 9 SIMLR clusters
- Rejected clusters (8 of 9 in the breast cancer Visium example) show QCC and/or QCM below 0.5 for the majority of cells
- Per-cluster metrics are computed after training one SCA model per cluster on raw counts, not on normalized/log-transformed data
- Reproducibility check: rerunning with the same dockerized rCASC images should reproduce the same set of passing clusters (computation reproducibility per Sandve rules)

## Limitations

- SCA reconstructs only specific clusters, so clusters failing the QCC/QCM > 0.5 threshold cannot be functionally annotated via the SCA latent space; absence of a passing cluster means no TF-ranking step is justified
- QCC/QCM evaluate reconstruction quality only; they do not themselves identify the biological drivers (TF ranking requires a separate COMET step)
- SCA usage can be challenging for users lacking strong computation skills; it requires a 64-bit Linux machine with Docker daemon installed
- Transcription factors and kinases are lowly expressed genes notably affected by dropout, which can affect the latent space the metrics are computed on

## Evidence

- [abstract] QCC/QCM metrics defined: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] Robustness metrics are a core implementation feature: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [readme] Docker-based reproducible execution requirement: [evidence span withheld — anchor status: not_found]
