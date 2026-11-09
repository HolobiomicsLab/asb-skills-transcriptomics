---
name: transcription-factor-activity-inference-from-autoencoder-latent-space
description: Use when a counts table with pre-defined cell/spot clusters (e.g., SIMLR
  annotations over a Visium breast cancer section) must be linked to specific transcription
  factors via an autoencoder latent space.
when_to_use_negative:
- The input is not a raw scRNAseq/spatial counts table with a pre-computed cluster
  assignment — clustering must be done first (e.g., with SIMLR) before SCA can query
  subpopulations
- The goal is de novo clustering rather than functional annotation of existing clusters;
  SCA reconstructs only specific clusters and is not a general clustering tool
- 'You need TF-miRNA circuit measurements: miRNAs cannot currently be quantified together
  with mRNAs at single-cell level, so such interactions cannot be directly assessed'
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Encodes raw counts into a TF-target-informed latent space per cluster and
    yields QCC/QCM reconstruction metrics
  repo: github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework in which SCA is implemented as a module (with GUI), including the
    'autoencoder' function and reproducible dockerized execution
  repo: github.com/kendomaniac/rCASC
- name: SIMLR
  role: Provides the core clustering of the Visium section into 9 clusters that SCA
    then queries
- name: COMET (cometsc)
  role: Ranks transcription factors from the SCA latent-space frequency table of each
    reconstructed cluster
- name: Docker
  role: Runs the dockerized SCA/rCASC tools (containers at docker.io/repbioinfo) to
    guarantee computational reproducibility
  repo: github.com/kendomaniac/docker4seq
- name: R
  role: Environment in which rCASC/SCA is installed (via devtools) and executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:42350d18817858e08df6c6eae7f0ee76e4e79430a964a66ce9f7ea6f3fba96b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - SIMLR
  - rCASC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# transcription-factor-activity-inference-from-autoencoder-latent-space

## Summary

Infers cluster-specific transcription factor activity by encoding scRNAseq/spatial counts through the rCASC Sparsely-Connected Autoencoder (SCA) with TF-target-informed hidden layers, then ranking TFs in the latent space (e.g., with COMET). Applied on a Visium breast cancer section, it recovered SOX5 as the top TF for cluster 7 (COMETsc 2.88E-184, TP 0.623, TN 0.968).

## When to use

Use when a counts table with pre-defined cell/spot clusters (e.g., SIMLR annotations over a Visium breast cancer section) must be linked to specific transcription factors via an autoencoder latent space.

## When NOT to use

- The input is not a raw scRNAseq/spatial counts table with a pre-computed cluster assignment — clustering must be done first (e.g., with SIMLR) before SCA can query subpopulations
- The goal is de novo clustering rather than functional annotation of existing clusters; SCA reconstructs only specific clusters and is not a general clustering tool
- You need TF-miRNA circuit measurements: miRNAs cannot currently be quantified together with mRNAs at single-cell level, so such interactions cannot be directly assessed

## Inputs

- Raw UMI counts table (e.g., Visium breast cancer Block A Section 1 spatial transcriptomics counts)
- Cluster annotation assigning each spot/cell to a cluster (e.g., SIMLR 9-cluster labels from rCASC)
- Prior knowledge of TF targets (optionally miRNA, kinase, cancer-related immune signatures) defining the sparsely-connected hidden layers

## Outputs

- Per-cluster QCC and QCM quality-control scores (threshold > 0.5) identifying which clusters are reconstructed
- Ranked list of transcription factors per reconstructed cluster with COMETsc statistics, TP and TN values (e.g., SOX5, COMETsc 2.88E-184, TP 0.623, TN 0.968)

## How to apply

Load the raw counts table and the cluster annotation (e.g., SIMLR assigning Visium Block A Section 1 spots to 9 clusters) — raw counts are the SCA input. Train the rCASC SCA autoencoder, whose hidden layers are sparsely connected according to experimentally validated TF-target (and optionally miRNA, kinase, immune-signature) priors, once per cluster. Compute the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics per cluster and accept only clusters where QCC and QCM exceed 0.5 for the majority of cells, since SCA by design reconstructs only the clusters where the encoding space drives cell aggregation. For each accepted cluster, run COMET (cometsc) on the SCA-encoded-space frequency table to rank TFs; validate by confirming the expected top TF and statistics (here: SOX5, COMETsc 2.88E-184, TP 0.623, TN 0.968, uniquely for cluster 7).

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Encodes raw counts into a TF-target-informed latent space per cluster and yields QCC/QCM reconstruction metrics) — github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework in which SCA is implemented as a module (with GUI), including the 'autoencoder' function and reproducible dockerized execution) — github.com/kendomaniac/rCASC
- **SIMLR** (Provides the core clustering of the Visium section into 9 clusters that SCA then queries)
- **COMET (cometsc)** (Ranks transcription factors from the SCA latent-space frequency table of each reconstructed cluster)
- **Docker** (Runs the dockerized SCA/rCASC tools (containers at docker.io/repbioinfo) to guarantee computational reproducibility) — github.com/kendomaniac/docker4seq
- **R** (Environment in which rCASC/SCA is installed (via devtools) and executed)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCC and QCM must both exceed 0.5 for the majority of cells in a cluster; on the Visium breast cancer section only cluster 7 passed
- COMET on the reconstructed cluster's latent space must list SOX5 as top-ranked TF with COMETsc = 2.88E-184, TP = 0.623, TN = 0.968
- Clusters failing the QCC/QCM > 0.5 threshold should be discarded rather than force-interpreted, matching SCA's design to reconstruct only clusters where the encoding space drives aggregation
- Reproducibility check: the exact docker containers (docker.io/repbioinfo) and rCASC version used must be tracked per Sandve-style reproducibility rules

## Limitations

- Transcription factors and kinases are lowly expressed genes and can be notably affected by dropout, which can weaken TF ranking
- scRNAseq is very noisy, and SCA reconstruction is selective: only clusters where the SCA encoding space is key to cell aggregation are recovered (here only 1 of 9 clusters)
- SCA usage can be challenging for life-science personnel lacking strong computation skills, despite the GUI; the workflow requires Docker on a 64-bit Linux machine

## Evidence

- [abstract] QCC/QCM threshold and reconstruction quality: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA selectively reconstructs clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] Rationale for TF identification via SCA: "their results suggest that SCA can be efficiently exploited in the identification of transcription factors with differential activity between conditions or cell types."
- [discussion] Hidden layers from validated priors: "derived by experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell cluster-specific"
- [readme] Dockerized reproducible execution: [evidence span withheld — anchor status: not_found]
- [abstract] SCA as rCASC module: "SCA analysis is implemented as module in rCASC framework and it is supported by a GUI"
