---
name: autoencoder-quality-metric-evaluation-qcm-qcc
description: Use when quantifying whether an SCA/SSCA latent space (built from TF,
  miRNA, kinase targets, or immune signatures) reconstructs a given single-cell cluster,
  or when comparing knowledge-group combinations (e.g., TF+miRNA vs TF alone) by QCM/QCC
  on a counts table with pre-computed clusters.
when_to_use_negative:
- Input lacks pre-computed cluster assignments — SCA/QCM/QCC evaluate reconstruction
  of defined clusters, not perform the initial clustering itself
- Goal is general-purpose benchmarking of arbitrary (non-knowledge-guided) autoencoders
  — QCM/QCC are specific to SCA's sparsely-connected, knowledge-derived encoded space
- Single-cell data exceed ~80% zero counts such that normalization tools like SCnorm
  cannot converge (per SCnorm README), making upstream counts unreliable for SCA encoding
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Reconstructs clusters from knowledge-derived encoded spaces; its bundled metrics
    QCM and QCC quantify reconstruction quality
  repo: https://github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework hosting the SCA module (with GUI) that runs the cluster reconstruction
    and computes QCM/QCC
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Containers (docker.io/repbioinfo) guarantee reproducible execution of the
    SCA/rCASC workflow
- name: R
  role: Environment in which rCASC/SCA is installed and run (install_github("kendomaniac/rCASC"))
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:c9710b205826e64acdf3b12650f9c25105dd083da401227a65af281fd3c05984
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# autoencoder-quality-metric-evaluation-qcm-qcc

## Summary

Evaluate the quality of Sparsely-Connected Autoencoder (SCA/SSCA) cluster reconstruction on scRNAseq data using the two bespoke metrics QCC (Quality Control of Cluster) and QCM (Quality Control of Model), with a significance threshold of mean QCM and QCC ≥ 0.5 marking clusters successfully reconstructed.

## When to use

Use when quantifying whether an SCA/SSCA latent space (built from TF, miRNA, kinase targets, or immune signatures) reconstructs a given single-cell cluster, or when comparing knowledge-group combinations (e.g., TF+miRNA vs TF alone) by QCM/QCC on a counts table with pre-computed clusters.

## When NOT to use

- Input lacks pre-computed cluster assignments — SCA/QCM/QCC evaluate reconstruction of defined clusters, not perform the initial clustering itself
- Goal is general-purpose benchmarking of arbitrary (non-knowledge-guided) autoencoders — QCM/QCC are specific to SCA's sparsely-connected, knowledge-derived encoded space
- Single-cell data exceed ~80% zero counts such that normalization tools like SCnorm cannot converge (per SCnorm README), making upstream counts unreliable for SCA encoding

## Inputs

- Single-cell UMI counts table with pre-computed cell-cluster assignments (e.g., setA clusters 1-5: B cells, monocytes, HSC, naïve T cells)
- Latent-space definition files: experimentally validated TF targets, miRNA targets, kinase targets, and cancer-related immune signatures
- rCASC/Docker environment with the SCA module installed

## Outputs

- Per-run QCM and QCC values for each cluster and each latent-space configuration
- Cluster significance calls (significant if mean QCM and QCC ≥ 0.5)
- Summary table, e.g., table1_ssca_qcm_qcc.csv mirroring Table 1 / Fig. 5 of the paper

## How to apply

Run the SCA module in rCASC (via its Docker containers) on the setA single-cell counts table with pre-computed cell-cluster assignments, once per latent-space configuration: single knowledge groups (TF-only, miRNA-only, IS-only, kinase-only) as baselines and combinations (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) for SSCA. For each run, compute QCM (Quality Control of Model, robustness of the neural network encoding) and QCC (Quality Control of Cluster, ability to reconstruct a specific cell cluster), and flag a cluster as significant when its mean QCM and QCC are ≥0.5. Compare each combined latent space against the single-group baselines: the expected pattern is cluster 3 (HSC, marker miR-129-2-3P) entering the significant area only with TF+miRNA, clusters 4/5 entering with TF+miRNA+IS, and no cluster significant with kinase alone. Assemble all QCM/QCC values and significance calls into a results table (e.g., table1_ssca_qcm_qcc.csv mirroring Table 1/Fig. 5).

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Reconstructs clusters from knowledge-derived encoded spaces; its bundled metrics QCM and QCC quantify reconstruction quality) — https://github.com/kendomaniac/rCASC
- **rCASC** (Framework hosting the SCA module (with GUI) that runs the cluster reconstruction and computes QCM/QCC) — https://github.com/kendomaniac/rCASC
- **Docker** (Containers (docker.io/repbioinfo) guarantee reproducible execution of the SCA/rCASC workflow)
- **R** (Environment in which rCASC/SCA is installed and run (install_github("kendomaniac/rCASC"))) — https://github.com/kendomaniac/rCASC

## Examples

```
library(devtools); install_github("kendomaniac/rCASC"); # then run SCA module per latent space and collect QCM/QCC per cluster
```

## Evaluation signals

- A cluster is judged significant only when its mean QCM and QCC are both ≥ 0.5
- Reproduced significance pattern: cluster 3 (HSC) significant under TF+miRNA; clusters 4/5 significant under TF+miRNA+IS; no cluster significant under kinase alone
- Combined latent spaces (TF+miRNA, TF+miRNA+IS) show higher QCM/QCC than each individual knowledge group (TF-only, miRNA-only, IS-only, kinase-only baselines)
- Cluster-specific marker evidence supports the calls (e.g., miR-129-2-3P for HSC cluster 3; POU2F2 negation consistent with naïve T cell cluster 5)
- Output table (table1_ssca_qcm_qcc.csv) contains QCM/QCC values for all four single-group and three combined configurations across clusters 1-5

## Limitations

- QCM/QCC quantify SCA-specific encoding robustness and cluster reconstruction; the paper notes prior work 'did not report a well-defined scoring metric to evaluate the efficacy of SCA', so these metrics are not transferable to generic autoencoders
- Transcription factors and kinases are lowly expressed genes notably affected by dropout, which can degrade latent-space quality for those knowledge groups
- It is not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits are inferred via miRNA target-gene latent spaces rather than direct measurement
- Kinase-only latent space brought no cluster into the significant area, so knowledge-group choice strongly constrains which clusters can be reconstructed

## Evidence

- [abstract] We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters where the encoding space is key to aggregation: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] SSCA combines TF, miRNA, IS, Kinases in a unique latent space: "combines different biological knowledge, i.e., TFs, miRNAs, IS, Kinases, in a unique latent space"
- [discussion] Hidden layers from experimentally validated data grasp cluster-specific features: "derived by experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell cluster-specific"
- [readme] README: SCA tools embedded in Docker containers for reproducibility: [evidence span withheld — anchor status: not_found]
- [methods] Article methods: rCASC installation in R via devtools: "install.packages("devtools")"
