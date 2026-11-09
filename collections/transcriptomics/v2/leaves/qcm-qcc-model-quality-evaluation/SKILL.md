---
name: qcm-qcc-model-quality-evaluation
description: Use when a sparsely-connected autoencoder analysis of scRNAseq clusters
  needs a quantitative verdict on whether the autoencoder model and a given cluster's
  reconstruction are trustworthy (e.g., verifying QCM/QCC > 0.5 for NK and naive T
  cell clusters of setA).
when_to_use_negative:
- No clustering of scRNAseq data has been performed yet, so there are no cell subpopulations
  to evaluate reconstruction quality for
- The autoencoder's hidden layers are not derived from experimentally validated data
  (TF targets, miRNA targets, kinase targets, cancer-related immune signatures), so
  QCM/QCC acceptance does not speak to functional support of clusters
- The goal is general clustering performance benchmarking of non-neural-network clustering
  tools rather than judging SCA's reconstruction of a specific cluster in its encoded
  space
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Provides the QCM and QCC metrics that quantify the ability of SCA to reconstruct
    valuable cell clusters
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework in which SCA is implemented as a module (autoencoder4clustering,
    autoencoderAnalysis4Clustering) with GUI support, executed in Docker containers
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Daemon required to run the rCASC/SCA workflow containers that produce the
    QCM/QCC outputs
- name: R
  role: Environment in which rCASC and its SCA modules are installed and run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:c6e6e25ed387c2282d39ded679a9500d196649f83ce0c4c806e022304b9024cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# qcm-qcc-model-quality-evaluation

## Summary

Use the QCM (Quality Control of Model) and QCC (Quality Control of Cluster) metrics from the Sparsely-Connected Autoencoder (SCA) rCASC module to quantify whether the SCA encoded space (e.g., built from cancer-related immune signatures) reliably reconstructs specific single-cell clusters, with values > 0.5 indicating cluster-level support.

## When to use

Use when a sparsely-connected autoencoder analysis of scRNAseq clusters needs a quantitative verdict on whether the autoencoder model and a given cluster's reconstruction are trustworthy (e.g., verifying QCM/QCC > 0.5 for NK and naive T cell clusters of setA).

## When NOT to use

- No clustering of scRNAseq data has been performed yet, so there are no cell subpopulations to evaluate reconstruction quality for
- The autoencoder's hidden layers are not derived from experimentally validated data (TF targets, miRNA targets, kinase targets, cancer-related immune signatures), so QCM/QCC acceptance does not speak to functional support of clusters
- The goal is general clustering performance benchmarking of non-neural-network clustering tools rather than judging SCA's reconstruction of a specific cluster in its encoded space

## Inputs

- scRNAseq counts matrix (e.g., setA blood-cell dataset) with cluster annotations
- SCA autoencoder output from rCASC (autoencoder4clustering / autoencoderAnalysis4Clustering) run in Docker containers
- Latent space definition (e.g., manually curated cancer-related immune signatures) used to encode hidden layers

## Outputs

- Per-cell and per-cluster QCM (Quality Control of Model) scores
- Per-cell and per-cluster QCC (Quality Control of Cluster) scores
- Validation table combining QCM/QCC scores with COMET (cometsc) gene-set enrichment p-values for accepted clusters

## How to apply

After training SCA (autoencoder4clustering followed by autoencoderAnalysis4Clustering) inside its reproducible Docker containers on a counts matrix with clustering annotation, compute QCM and QCC from the rCASC SCA output. QCM estimates the robustness/quality of the autoencoder model itself, while QCC quantifies the ability of SCA to reconstruct that specific cell cluster; the rationale is that SCA should reconstruct only clusters where the encoding space (TF targets, miRNA targets, kinase targets, or cancer-related immune signatures) is a key element of cell aggregation. Accept a cluster only when both metrics exceed 0.5 for the majority of its cells (e.g., clusters 4/NK and 5/naive T cells in the setA blood-cell dataset under the immune-signature latent space). Assemble the per-cluster QCM/QCC scores into a validation table together with downstream COMET enrichment p-values (e.g., NK signature 5.75E-54 for cluster 4, ASTHMA KEGG negation 5.35E-84 for cluster 5) as the final evidence of a supported cluster.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Provides the QCM and QCC metrics that quantify the ability of SCA to reconstruct valuable cell clusters) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework in which SCA is implemented as a module (autoencoder4clustering, autoencoderAnalysis4Clustering) with GUI support, executed in Docker containers) — https://github.com/kendomaniac/rCASC
- **Docker** (Daemon required to run the rCASC/SCA workflow containers that produce the QCM/QCC outputs)
- **R** (Environment in which rCASC and its SCA modules are installed and run)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCM and QCC must both exceed 0.5 for the majority of cells in the target cluster (e.g., clusters 4 and 5 of setA)
- The accepted clusters should be exactly those where SCA reconstructs cells, i.e., clusters where the encoding space is key to aggregation
- COMET (cometsc) enrichment on accepted clusters must reproduce reported values, e.g., NK signature p-value 5.75E-54 for cluster 4 and ASTHMA KEGG negation p-value 5.35E-84 for cluster 5
- The run was executed inside the pinned rCASC Docker containers (docker.io/repbioinfo) to guarantee computational reproducibility of the metrics
- QCM/QCC scores are assembled into a single validation table that matches the workflow's acceptance criteria before the result is accepted

## Limitations

- SCA usage can be challenging for life science and medical personnel lacking strong computational skills
- scRNAseq is noisy and affected by dropout (more zeros than expected, bias for poorly expressed genes), and transcription factors and kinases are lowly expressed and notably affected by dropout
- SCA reconstructs only specific clusters, so QCM/QCC > 0.5 means the encoding space supports those clusters only, not all clusters in the dataset

## Evidence

- [abstract] QCC and QCM defined as dual quality metrics: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only supported clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] Metrics estimate encoding robustness: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [readme] Dockerized reproducible execution: [evidence span withheld — anchor status: not_found]
- [discussion] Hidden layers from validated signatures: "derived by experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell cluster-specific"
