---
name: progressive-biological-knowledge-integration-analysis
description: Decision support for choosing which combination of biological knowledge
  (TF, miRNA, IS, Kinase) to embed in an SCA/SSCA latent space when single-knowledge
  spaces fail to reconstruct specific single-cell clusters.
when_to_use_negative:
- When the goal is cluster discovery itself rather than functional reconstruction
  of already-defined clusters — SCA reconstructs only specific clusters of a pre-existing
  clustering.
- When no pre-computed cluster assignments (e.g. clusters 1-5 of setA) are available
  to evaluate QCC/QCM against.
- When working with data where miRNAs must be directly quantified alongside mRNAs
  — it is currently not possible to quantify miRNAs together with mRNAs at single-cell
  level, so TF-miRNA circuits cannot be directly measured.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Runs the autoencoder with single and combined biological-knowledge latent
    spaces to reconstruct clusters and derive the encoding evaluated by QCM/QCC
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework embedding SCA/SSCA as a module (with GUI) used to execute the analysis
    reproducibly in Docker containers
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Hosts the rCASC/SCA containers (docker.io/repbioinfo) guaranteeing computation
    reproducibility of the SSCA runs
- name: R
  role: Environment in which rCASC, devtools, and the SSCA workflow are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:3095e4f19789b183354e8cad24c8189b54b0d086f0b70f1ad0c5946b0f8ffc58
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# progressive-biological-knowledge-integration-analysis

## Summary

Progressively combines biological knowledge groups (experimentally validated TF targets, miRNA targets, kinase targets, and cancer-related immune signatures) into single SSCA latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) and evaluates whether combined spaces improve cluster reconstruction (QCM/QCC) on the setA blood-cell scRNAseq dataset versus each knowledge group alone.

## When to use

Decision support for choosing which combination of biological knowledge (TF, miRNA, IS, Kinase) to embed in an SCA/SSCA latent space when single-knowledge spaces fail to reconstruct specific single-cell clusters.

## When NOT to use

- When the goal is cluster discovery itself rather than functional reconstruction of already-defined clusters — SCA reconstructs only specific clusters of a pre-existing clustering.
- When no pre-computed cluster assignments (e.g. clusters 1-5 of setA) are available to evaluate QCC/QCM against.
- When working with data where miRNAs must be directly quantified alongside mRNAs — it is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured.

## Inputs

- setA single-cell RNAseq counts table with pre-computed cell-cluster assignments (clusters 1-5: B cells, monocytes, HSC, naïve T cell)
- Experimentally validated transcription factor (TF) target sets
- Experimentally validated miRNA target sets (e.g. miR-129-2-3P evidence for HSC)
- Cancer-related immune signature (IS) sets
- Kinase target sets

## Outputs

- table1_ssca_qcm_qcc.csv with QCM/QCC values per latent-space combination and per cluster
- Cluster-significance calls (clusters entering the significant reconstruction area, mean QCM/QCC ≥ 0.5)
- Per-cluster marker evidence (e.g. miR-129-2-3P for HSC; POU2F2 negation for naïve T cell cluster 5)

## How to apply

Load the setA scRNAseq counts table with pre-computed cell-cluster assignments (clusters 1-5: B cells, monocytes, HSC, naïve T cell) in R within the rCASC/Docker environment. Establish baselines by running SCA separately with TF-only, miRNA-only, IS-only, and kinase-only latent spaces, recording QCM and QCC for each (kinase alone brings no cluster into the significant area). Then run SSCA with combined latent spaces TF+miRNA, TF+miRNA+IS, and TF+miRNA+IS+Kinase, each merging experimentally validated target/signature sets into one encoded space. Compute QCC (Quality Control of Cluster) and QCM (Quality Control of Model) per combination and identify clusters entering the significant area (mean QCM and QCC ≥ 0.5): cluster 3 (HSC, marker miR-129-2-3P) under TF+miRNA, and clusters 4/5 under TF+miRNA+IS. Compare each combination against single-group baselines to confirm progressive improvement, and assemble QCM/QCC values plus per-cluster marker evidence into a table mirroring Table 1 / Fig. 5 of the paper.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Runs the autoencoder with single and combined biological-knowledge latent spaces to reconstruct clusters and derive the encoding evaluated by QCM/QCC) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework embedding SCA/SSCA as a module (with GUI) used to execute the analysis reproducibly in Docker containers) — https://github.com/kendomaniac/rCASC
- **Docker** (Hosts the rCASC/SCA containers (docker.io/repbioinfo) guaranteeing computation reproducibility of the SSCA runs)
- **R** (Environment in which rCASC, devtools, and the SSCA workflow are executed)

## Examples

```
library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Combined latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) show higher QCM/QCC than each individual knowledge-group baseline.
- Cluster 3 (HSC) reaches the significant area (mean QCM and QCC ≥ 0.5) under TF+miRNA, which was not achievable with miRNA targets alone.
- Clusters 4/5 enter the significant area under TF+miRNA+IS.
- No cluster is significant with kinase targets alone (negative baseline control).
- Output table1_ssca_qcm_qcc.csv reproduces the cluster-significance pattern and per-cluster markers (e.g. miR-129-2-3P for HSC) mirroring Table 1 / Fig. 5 of the paper.

## Limitations

- Kinase targets alone bring no cluster into the significant reconstruction area, so kinase-only latent spaces are not useful for this dataset.
- Transcription factors and kinases are lowly expressed genes and can be notably affected by scRNAseq dropout.
- scRNAseq data are very noisy and produce more zeros than expected, with greater bias for poorly expressed genes.
- Prior SCA work (Gold and coworkers) provided no implementation or well-defined scoring metric; only this rCASC/SCA implementation with QCM/QCC makes the efficacy evaluable.

## Evidence

- [abstract] QCC and QCM metrics quantify SCA cluster reconstruction: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] SSCA combines multiple knowledge groups in one latent space: "combines different biological knowledge, i.e., TFs, miRNAs, IS, Kinases, in a unique latent space"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] SCA encoded space derived from experimentally validated data: "derived by experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell cluster-specific"
- [discussion] SCA available via rCASC GUI within Docker containers: "our implementation of SCA within rCASC framework9 solves the above issue, because SCA is also fully accessible via GUI."
- [readme] SCA workflow containers embedded for reproducibility: [evidence span withheld — anchor status: not_found]
