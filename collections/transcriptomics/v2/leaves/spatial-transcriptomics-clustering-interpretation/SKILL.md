---
name: spatial-transcriptomics-clustering-interpretation
description: Use when a spatial transcriptomics dataset (e.g., Visium counts table)
  already has spot-to-cluster annotations (e.g., from SIMLR) and the goal is to test
  which clusters encode interpretable transcription-factor programs.
when_to_use_negative:
- The input is raw FASTQ without a counts table or an existing cluster annotation
  — pre-processing (e.g., stpipeline) and clustering (e.g., SIMLR) must happen first
- The goal is a general functional annotation not tied to cluster reconstruction quality
  — SCA deliberately reconstructs only specific clusters, so clusters failing QCC/QCM
  > 0.5 should not be forced through COMET interpretation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: rCASC
  role: Framework hosting the SCA autoencoder module (with GUI) used to reconstruct
    spatial clusters and compute QCC/QCM
  repo: https://github.com/kendomaniac/rCASC
- name: SCA (Sparsely-Connected Autoencoder)
  role: Encodes each cluster's raw counts into a TF-target-informed latent space for
    functional mining
  repo: https://github.com/kendomaniac/SCAtutorial
- name: SIMLR
  role: Provides the spot-to-cluster annotation (9 clusters) that defines the clusters
    submitted to SCA
- name: COMET
  role: Ranks transcription factors from the SCA latent-space frequency table (cometsc)
    to identify cluster-specific TFs
- name: Docker
  role: Runs the rCASC/SCA workflow containers (docker.io/repbioinfo) for reproducibility
  repo: https://github.com/kendomaniac/rCASC
- name: R
  role: Environment in which rCASC, SCA, and the workflow functions are installed
    and executed
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:56240a631a24ccaf987b2e2f87c9182f839d3e6e6b432beb1736760edc2499bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - COMET
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - SIMLR
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spatial-transcriptomics-clustering-interpretation

## Summary

Interprets clusters of a spatially resolved transcriptomics section (e.g., a Visium breast cancer Block A Section 1) by reconstructing each cluster with a TF-target-informed Sparsely-Connected Autoencoder (SCA, an rCASC module) and ranking cluster-specific transcription factors from the SCA latent space with COMET. It recovers which histological clusters carry identifiable TF-driven functional identity rather than just spatial partitions.

## When to use

Use when a spatial transcriptomics dataset (e.g., Visium counts table) already has spot-to-cluster annotations (e.g., from SIMLR) and the goal is to test which clusters encode interpretable transcription-factor programs.

## When NOT to use

- The input is raw FASTQ without a counts table or an existing cluster annotation — pre-processing (e.g., stpipeline) and clustering (e.g., SIMLR) must happen first
- The goal is a general functional annotation not tied to cluster reconstruction quality — SCA deliberately reconstructs only specific clusters, so clusters failing QCC/QCM > 0.5 should not be forced through COMET interpretation

## Inputs

- Raw counts table for a spatial transcriptomics section (e.g., Visium breast cancer Block A Section 1)
- Spot-to-cluster annotation file (e.g., SIMLR cluster labels, 9 clusters)
- TF-target experimentally validated regulogs used to sparsely connect SCA hidden layers

## Outputs

- Per-cluster QCC and QCM quality metrics with a > 0.5 pass/fail threshold
- COMET (cometsc) ranked transcription factor table per reconstructed cluster (COMETsc statistic, TP, TN)
- Identified cluster-specific TF signature (e.g., SOX5 specific for cluster 7)

## How to apply

Load the raw counts table for the spatial section together with the clustering annotation (e.g., SIMLR assigning spots to 9 clusters); raw counts are used as SCA input. Train the rCASC SCA autoencoder, whose hidden layers are sparsely connected according to experimentally validated TF-target regulogs, separately for each cluster. Compute the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics per cluster and accept only clusters where QCC and QCM exceed 0.5 for the majority of cells — SCA is expected to reconstruct only specific clusters, which is itself diagnostic of where the encoding space drives cell aggregation. For each accepted cluster, run COMET (cometsc) on the SCA-encoded space frequency table to rank transcription factors, and report the top TF with its COMETsc statistic, TP, and TN values. Validation: only the cluster(s) passing QCC/QCM > 0.5 are interpreted, and the top-ranked TF (e.g., SOX5 for cluster 7, COMETsc 2.88E-184, TP 0.623, TN 0.968) must appear at the top of the COMET table.

## Related tools

- **rCASC** (Framework hosting the SCA autoencoder module (with GUI) used to reconstruct spatial clusters and compute QCC/QCM) — https://github.com/kendomaniac/rCASC
- **SCA (Sparsely-Connected Autoencoder)** (Encodes each cluster's raw counts into a TF-target-informed latent space for functional mining) — https://github.com/kendomaniac/SCAtutorial
- **SIMLR** (Provides the spot-to-cluster annotation (9 clusters) that defines the clusters submitted to SCA)
- **COMET** (Ranks transcription factors from the SCA latent-space frequency table (cometsc) to identify cluster-specific TFs)
- **Docker** (Runs the rCASC/SCA workflow containers (docker.io/repbioinfo) for reproducibility) — https://github.com/kendomaniac/rCASC
- **R** (Environment in which rCASC, SCA, and the workflow functions are installed and executed) — https://github.com/kendomaniac/rCASC

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC"); library(rCASC)
```

## Evaluation signals

- QCC and QCM must exceed 0.5 for the majority of cells in the accepted cluster (e.g., cluster 7) and ideally only that cluster
- The COMET table must list SOX5 as the top-ranked TF with COMETsc ≈ 2.88E-184, TP = 0.623, TN = 0.968
- SCA reconstructs only specific clusters — clusters that are all reconstructed or all rejected indicate a problem with the counts input or encoding space
- The latent space must be built from TF-target-informed sparsely-connected hidden layers, not a fully connected autoencoder
- Raw (not log-transformed/normalized) counts are used as SCA input, per the published workflow

## Limitations

- Only some clusters are reconstructed by SCA; clusters failing QCC/QCM > 0.5 yield no interpretable TF signature and cannot be forced through the COMET step
- Transcription factors are lowly expressed genes and can be notably affected by scRNA-seq dropout, which limits TF detection
- SCA usage can be challenging for users lacking strong computation skills, though the rCASC GUI mitigates this; the workflow requires a 64-bit Linux machine with the Docker daemon installed

## Evidence

- [abstract] QCC/QCM metrics: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] cluster selectivity: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] TF identification rationale: "their results suggest that SCA can be efficiently exploited in the identification of transcription factors with differential activity between conditions or cell types."
- [discussion] dropout caveat for TFs: "Transcription factors and kinases are encoded by genes characterized by a relatively low expression in cells, thus they can be notably affected by dropout."
- [readme] rCASC availability: [evidence span withheld — anchor status: not_found]
