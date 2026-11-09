---
name: single-cell-autoencoder-latent-space-training
description: 'Use when you need to functionally mine scRNAseq clusters with a knowledge-biased
  autoencoder: you have a filtered, log10-transformed counts table plus reference
  cluster assignments and want per-cluster reconstruction quality (QCC/QCM) and TF
  signatures.'
when_to_use_negative:
- You only need generic dimensionality reduction or visualization without functional
  annotation — SCA reconstructs only clusters where the encoding space is key to cell
  aggregation, so many clusters will legitimately fail QCC/QCM >= 0.5.
- The dataset has not been filtered/normalized (e.g., no filterZeros/lorenzFilter
  and no counts2log transformation applied), since the autoencoder expects a filtered,
  log10-transformed table.
- No Docker daemon is available on a 64-bit Linux machine, as the rCASC SCA modules
  are embedded in docker.io/repbioinfo containers.
- Your goal is single-cluster-only benchmarking without a reference clustering — QCC/QCM
  evaluation requires reference cluster assignments to compare against.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Knowledge-biased autoencoder whose sparse hidden layers encode the TF-target
    (TRRUST) latent space; reconstructs clusters evaluated by QCC/QCM
  repo: github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the SCA modules (autoencoder, autoencoderAnalysis, clusterNgriph,
    counts2log), reproducible via Docker containers
  repo: github.com/kendomaniac/rCASC
- name: Docker
  role: Docker daemon required to execute rCASC SCA functions (group='docker', scratch.folder)
    in reproducible containers
- name: R
  role: Environment in which rCASC is installed (install.packages('devtools'); install_github('kendomaniac/rCASC'))
    and the SCA workflow is run
  repo: github.com/kendomaniac/rCASC
- name: devtools
  role: Used to install rCASC from GitHub
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:cbb63c17ba4703e6b131a1e16a890efd2bf64ba406c8e985023795e39678e589
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - R
  - SCA
  - SCA (Sparsely-Connected Autoencoder)
  - autoencoder
  - autoencoderAnalysis
  - clusterNgriph
  - counts2log
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# single-cell-autoencoder-latent-space-training

## Summary

Train a Sparsely-Connected Autoencoder (SCA) whose hidden layers are biased by experimentally validated latent-space priors (TRRUST TF targets, miRNA targets, kinase targets, immune signatures) on single-cell RNA-seq count data, repeated over multiple seeds, to reconstruct functional cell clusters scored by QCC and QCM.

## When to use

Use when you need to functionally mine scRNAseq clusters with a knowledge-biased autoencoder: you have a filtered, log10-transformed counts table plus reference cluster assignments and want per-cluster reconstruction quality (QCC/QCM) and TF signatures.

## When NOT to use

- You only need generic dimensionality reduction or visualization without functional annotation — SCA reconstructs only clusters where the encoding space is key to cell aggregation, so many clusters will legitimately fail QCC/QCM >= 0.5.
- The dataset has not been filtered/normalized (e.g., no filterZeros/lorenzFilter and no counts2log transformation applied), since the autoencoder expects a filtered, log10-transformed table.
- No Docker daemon is available on a 64-bit Linux machine, as the rCASC SCA modules are embedded in docker.io/repbioinfo containers.
- Your goal is single-cluster-only benchmarking without a reference clustering — QCC/QCM evaluation requires reference cluster assignments to compare against.

## Inputs

- single-cell UMI counts table (e.g., filtered setA PBMC counts)
- log10-transformed count table produced by counts2log
- reference cluster assignments (e.g., griph-derived cell clustering)
- TF-target latent-space prior (e.g., TRRUST)
- Docker daemon (repbioinfo containers) and scratch folder

## Outputs

- per-run SCA encoded space for each cell
- per-cluster QCC and QCM scores per run and aggregated means (e.g., sca_qcc_qcm_cluster_metrics.csv)
- top-ranked transcription factors (SSCA signatures) per reconstructed cluster

## How to apply

Load the single-cell counts table (e.g., the setA FACS-purified PBMC dataset) into the rCASC R environment executed through the Docker daemon, convert raw counts to log10 with counts2log, and take reference cluster labels from an established method such as griph (clusterNgriph). Train the SCA autoencoder (group='docker', scratch.folder set, regularization=TRUE, variational=FALSE, permutation enabled) with hidden layers biased by the TRRUST TF-target prior, repeating the encoding over multiple runs with distinct seeds to assess stability. For each run, run autoencoderAnalysis (group='docker', nCluster matching the reference clustering, separator, Sp parameter, bestPerm from the SCA runs) on the encoded space to compute per-cluster QCC (Quality Control of Cluster) and QCM (Quality Control of Model) scores, then aggregate mean per-cluster values across runs. The rationale is that SCA's efficacy comes from reconstructing only specific clusters — those where the SCA encoding space is key to cell aggregation — so a cluster is judged reconstructable when its mean QCC and QCM are >= 0.5 (e.g., in setA only clusters 1 and 2, B cells and monocytes, passed this threshold, matching Fig. 2b-c).

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Knowledge-biased autoencoder whose sparse hidden layers encode the TF-target (TRRUST) latent space; reconstructs clusters evaluated by QCC/QCM) — github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the SCA modules (autoencoder, autoencoderAnalysis, clusterNgriph, counts2log), reproducible via Docker containers) — github.com/kendomaniac/rCASC
- **Docker** (Docker daemon required to execute rCASC SCA functions (group='docker', scratch.folder) in reproducible containers)
- **R** (Environment in which rCASC is installed (install.packages('devtools'); install_github('kendomaniac/rCASC')) and the SCA workflow is run) — github.com/kendomaniac/rCASC
- **devtools** (Used to install rCASC from GitHub)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC"); autoencoder(group="docker", scratch.folder="/data/scratch", file=counts2log("setA_counts.csv"), nCluster=clusterNgriph_out, separator=",", Sp=1, regularization=TRUE, permutation=TRUE, seed=1234)
```

## Evaluation signals

- A cluster is considered reconstructed only if its mean QCC and mean QCM across multiple SCA runs (distinct seeds) are both >= 0.5.
- vSCA-vs-SCA check: variational variant should be rejected as non-improving if its QCM/QCC trajectories are nearly superimposable on plain SCA and top COMET markers match prior signatures (CHD4 for cluster 1, CEBPA for cluster 2).
- Top-ranked TFs derived from the SCA encoded space should be biologically consistent with the cluster identity (e.g., CHD4/CEBPA for B cells/monocytes in setA; POU2F2 negation fits naïve T cells in cluster 5).
- Reproducibility check per Sandve rules: distinct random seeds recorded per SCA run; the exact Docker images from docker.io/repbioinfo archived for the computation.
- nCluster passed to autoencoderAnalysis must match the griph reference cluster number, and bestPerm must come from the SCA permutation runs.

## Limitations

- SCA selectively reconstructs only some clusters: on setA only clusters 1 and 2 (B cells, monocytes) reached mean QCC/QCM >= 0.5, so failure on other clusters is expected behavior, not necessarily an error.
- Transcription factors and kinases are lowly expressed and notably affected by dropout; scRNAseq produces more zeros than expected, which can bias the encoding.
- SCA usage can be challenging for life-science and medical personnel lacking strong computation skills; running it requires a 64-bit Linux machine with Docker, and swarm (Docker Swarm) farm implementation was still under development.
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuit latent spaces cannot be directly measured.

## Evidence

- [abstract] QCC/QCM threshold selectivity: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [methods] docker requirement: "Docker daemon installed on the machine, for more info see this document"
- [methods] counts2log: "Converting a count table in log10: **counts2log**"
- [discussion] GUI accessibility: "our implementation of SCA within rCASC framework9 solves the above issue, because SCA is also fully accessible via GUI."
- [discussion] setA cluster 1 and 2 TFs: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [readme] docker containers and Linux requirement: [evidence span withheld — anchor status: not_found]
- [discussion] dropout limitation: "Transcription factors and kinases are encoded by genes characterized by a relatively low expression in cells, thus they can be notably affected by dropout."
