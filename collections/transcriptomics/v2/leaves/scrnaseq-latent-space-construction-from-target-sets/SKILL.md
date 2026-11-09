---
name: scrnaseq-latent-space-construction-from-target-sets
description: Use when a scRNAseq counts table with cell-cluster assignments (e.g.
  setA, clusters 1-5) must be functionally annotated by reconstructing clusters in
  an encoded space derived from validated target/signature sets rather than gene expression
  alone.
when_to_use_negative:
- The input is raw FASTQ or an unclustered counts table — clustering must be done
  first (e.g. via rCASC) before SCA can reconstruct defined subpopulations.
- The goal is generic dimensionality reduction or visualization of cells rather than
  testing which biological knowledge group functionally drives a specific cluster's
  aggregation.
- The dataset is miRNA+mRNA co-measurement — miRNAs cannot currently be quantified
  together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly
  measured.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: core autoencoder whose encoded space is derived from TF/miRNA/kinase target
    sets and immune signatures; provides QCM/QCC metrics
  repo: github.com/kendomaniac/rCASC
- name: rCASC
  role: framework/GUI in which SCA runs as a module, guaranteeing functional and computational
    reproducibility via Docker containers
  repo: github.com/kendomaniac/rCASC
- name: Docker
  role: encapsulates the SCA workflow tools in reproducible containers (docker.io/repbioinfo)
- name: R
  role: environment where rCASC and the SCA/SSCA functions are invoked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:6b3a4d73a3913dcf0f9f5ccac1fbc9e7ce5f611c7415d66aa07a0ab810447f99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scRNAseq-latent-space-construction-from-target-sets

## Summary

Build sparsely-connected autoencoder (SCA/SSCA) latent spaces for single-cell RNAseq from experimentally validated biological knowledge sets (TF targets, miRNA targets, kinase targets, cancer-related immune signatures), progressively combining groups so cluster-specific functional features drive cell aggregation, and score reconstruction quality with QCM/QCC.

## When to use

Use when a scRNAseq counts table with cell-cluster assignments (e.g. setA, clusters 1-5) must be functionally annotated by reconstructing clusters in an encoded space derived from validated target/signature sets rather than gene expression alone.

## When NOT to use

- The input is raw FASTQ or an unclustered counts table — clustering must be done first (e.g. via rCASC) before SCA can reconstruct defined subpopulations.
- The goal is generic dimensionality reduction or visualization of cells rather than testing which biological knowledge group functionally drives a specific cluster's aggregation.
- The dataset is miRNA+mRNA co-measurement — miRNAs cannot currently be quantified together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured.

## Inputs

- single-cell UMI counts table (e.g. setA) with pre-computed cell-cluster assignments (clusters 1-5: B cells, monocytes, HSC, naïve T cell)
- experimentally validated target/signature sets: TF targets, miRNA targets, kinase targets, cancer-related immune signatures

## Outputs

- per-configuration QCM and QCC scores for each cluster (single-group baselines and combined SSCA spaces)
- cluster-significance calls (clusters entering the significant area, mean QCM/QCC ≥0.5)
- per-cluster marker/signature evidence (e.g. miR-129-2-3P for HSC cluster 3, POU2F2 negation for naïve T cell cluster 5)
- summary table (e.g. table1_ssca_qcm_qcc.csv) mirroring Table 1 / Fig. 5

## How to apply

Load the counts table with pre-computed cluster assignments in R within the rCASC/Docker environment and run SCA separately with TF-only, miRNA-only, IS-only, and kinase-only latent spaces to record baseline QCM/QCC. Then run SSCA with combined latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase), each merging experimentally validated target sets into one encoded hidden layer. For each configuration compute QCC (Quality Control of Cluster) and QCM (Quality Control of Model), and call a cluster significant when mean QCM and QCC reach the significant reconstruction area (≥0.5). Compare each combined space against single-group baselines: on setA, TF+miRNA repositions cluster 3 (HSC, marker miR-129-2-3P) into the significant area while miRNA alone does not, and TF+miRNA+IS brings clusters 4/5 into significance, whereas kinase alone brings no cluster into the significant area. Assemble QCM/QCC values, significance calls, and per-cluster marker evidence (e.g. miR-129-2-3P for HSC) into a table mirroring Table 1 / Fig. 5.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (core autoencoder whose encoded space is derived from TF/miRNA/kinase target sets and immune signatures; provides QCM/QCC metrics) — github.com/kendomaniac/rCASC
- **rCASC** (framework/GUI in which SCA runs as a module, guaranteeing functional and computational reproducibility via Docker containers) — github.com/kendomaniac/rCASC
- **Docker** (encapsulates the SCA workflow tools in reproducible containers (docker.io/repbioinfo))
- **R** (environment where rCASC and the SCA/SSCA functions are invoked)

## Examples

```
library(devtools); install_github("kendomaniac/rCASC"); # then run the SCA/SSCA module on the setA counts table with TF+miRNA latent space and record QCM/QCC per cluster
```

## Evaluation signals

- Combined latent spaces (TF+miRNA, TF+miRNA+IS) must show higher QCM/QCC than each individual knowledge-group baseline.
- Cluster-significance pattern must be reproduced: cluster 3 (HSC) significant under TF+miRNA; clusters 4/5 significant under TF+miRNA+IS; no cluster significant with kinase alone.
- miRNA targets alone as latent space must fail to bring cluster 3 into the significant area, confirming the combined-space improvement.
- Significant clusters must have mean QCM and QCC ≥0.5 (significant reconstruction area).
- Output table rows must map each cluster to its supporting knowledge groups and marker evidence (e.g. miR-129-2-3P for HSC).

## Limitations

- SCA reconstructs only specific clusters — a cluster failing to enter the significant area means the encoding space is not a key element for that cluster's aggregation, not a tool error.
- Transcription factors and kinases are lowly expressed genes and can be notably affected by dropout in scRNA-seq.
- miRNAs cannot be quantified together with mRNAs at single-cell level, so TF-miRNA circuits are inferred from target sets rather than measured directly.
- Docker daemon required on a 64-bit Linux workstation/server; swarm-based farm implementation was still under development.

## Evidence

- [abstract] abstract: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [discussion] discussion: "combines different biological knowledge, i.e., TFs, miRNAs, IS, Kinases, in a unique latent space"
- [readme] readme: [evidence span withheld — anchor status: not_found]
