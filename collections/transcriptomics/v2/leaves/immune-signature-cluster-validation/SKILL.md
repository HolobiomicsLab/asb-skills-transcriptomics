---
name: immune-signature-cluster-validation
description: Use when single-cell clusters (e.g., setA clusters 4 NK cells and 5 naïve
  T cells) must be confirmed as supported by a cancer-immune-signature encoded space,
  requiring QCM/QCC > 0.5 and COMET p-values for signature/KEGG enrichment.
when_to_use_negative:
- Clusters other than the reference immune clusters (e.g., setA clusters 1 and 2,
  B cells and monocytes) for which no immune-signature support claim is being tested
- The goal is de novo clustering or cell type discovery rather than validating previously
  defined clusters against the signature encoding space
- No curated latent space (TF/miRNA/kinase/immune signatures) is available — SCA relies
  on experimentally validated data for its encoded space
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3934
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Reconstructs cell clusters in the cancer-immune-signature latent space and
    yields QCM/QCC reconstruction metrics for cluster support
  repo: github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework hosting the SCA modules (autoencoder4clustering, autoencoderAnalysis4Clustering
    with Sp neighbor-support scoring) executed in Docker containers
  repo: github.com/kendomaniac/rCASC
- name: COMET (cometsc)
  role: Gene-set enrichment of cluster-specific genes of clusters 4 and 5 against
    immune-signature and KEGG gene sets
- name: Docker
  role: Containers from docker.io/repbioinfo guarantee reproducible execution of the
    SCA workflow
  repo: github.com/kendomaniac/rCASC
- name: R
  role: Environment in which rCASC and the SCA modules are installed and driven
  repo: github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:da2a42915d63e1c9d5a45b773e44a8306b928772fa33159842e36d5e63cf51b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# immune-signature-cluster-validation

## Summary

Validates that specific scRNAseq cell clusters are functionally supported by a manually curated cancer-immune-signature latent space, using the SCA autoencoder's QCM/QCC reconstruction metrics (both must exceed 0.5) plus COMET (cometsc) gene-set enrichment of cluster-specific genes against immune-signature and KEGG gene sets.

## When to use

Use when single-cell clusters (e.g., setA clusters 4 NK cells and 5 naïve T cells) must be confirmed as supported by a cancer-immune-signature encoded space, requiring QCM/QCC > 0.5 and COMET p-values for signature/KEGG enrichment.

## When NOT to use

- Clusters other than the reference immune clusters (e.g., setA clusters 1 and 2, B cells and monocytes) for which no immune-signature support claim is being tested
- The goal is de novo clustering or cell type discovery rather than validating previously defined clusters against the signature encoding space
- No curated latent space (TF/miRNA/kinase/immune signatures) is available — SCA relies on experimentally validated data for its encoded space

## Inputs

- setA blood-cell scRNAseq counts matrix (UMI counts table)
- clustering annotation assigning cells to setA clusters (cluster 4 = Natural Killer cells, cluster 5 = Naïve T cells)
- manually curated cancer-immune-signature latent space
- immune-signature and KEGG gene sets for COMET enrichment

## Outputs

- QCM and QCC scores per cell for clusters 4 and 5 (accepted when both > 0.5)
- COMET (cometsc) enrichment p-values, e.g., NK signature 5.75E-54 for cluster 4 and ASTHMA KEGG negation 5.35E-84 for cluster 5
- single validation table assembling QCM/QCC scores and COMET p-values

## How to apply

Load the setA blood-cell scRNAseq counts matrix with its clustering annotation and select the reference clusters of interest (here clusters 4 and 5). Train the SCA model inside reproducible Docker containers via the rCASC module (autoencoder4clustering followed by autoencoderAnalysis4Clustering with Sp neighbor-support scoring), using the manually curated cancer immune-signature latent space as the encoded space. Compute the QCM (Quality Control of Model) and QCC (Quality Control of Cluster) metrics and accept the run only when both exceed 0.5 for the majority of cells in clusters 4 and 5, since these metrics quantify SCA's ability to reconstruct valuable cell clusters. Then run COMET gene-set enrichment (cometsc) on the cluster-specific genes against immune-signature and KEGG gene sets, and confirm the enrichment output matches the expected NK signature p-value 5.75E-54 for cluster 4 and the ASTHMA KEGG negation p-value 5.35E-84 for cluster 5. Assemble the QCM/QCC scores and COMET p-values into a single validation table as the final deliverable.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Reconstructs cell clusters in the cancer-immune-signature latent space and yields QCM/QCC reconstruction metrics for cluster support) — github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework hosting the SCA modules (autoencoder4clustering, autoencoderAnalysis4Clustering with Sp neighbor-support scoring) executed in Docker containers) — github.com/kendomaniac/rCASC
- **COMET (cometsc)** (Gene-set enrichment of cluster-specific genes of clusters 4 and 5 against immune-signature and KEGG gene sets)
- **Docker** (Containers from docker.io/repbioinfo guarantee reproducible execution of the SCA workflow) — github.com/kendomaniac/rCASC
- **R** (Environment in which rCASC and the SCA modules are installed and driven) — github.com/kendomaniac/rCASC

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCM and QCC both exceed 0.5 for the majority of cells in cluster 4 (NK) and cluster 5 (naïve T cells)
- COMET enrichment p-value for the NK signature on cluster 4 matches 5.75E-54
- COMET enrichment p-value for ASTHMA KEGG negation on cluster 5 matches 5.35E-84
- All SCA computations ran inside the pinned rCASC Docker containers (docker.io/repbioinfo), preserving computational reproducibility
- QCM/QCC scores and COMET p-values are assembled into a single validation table consistent with the acceptance rule

## Limitations

- SCA reconstructs only specific clusters — clusters where the encoding space is not a key element of cell aggregation will fail the QCM/QCC > 0.5 acceptance criterion
- scRNAseq is very noisy and dropout-prone; transcription factors and kinases are lowly expressed and notably affected by dropout, which can affect signature-based encoding
- The curated immune-signature latent space must already exist — it is a manually assembled input, not produced by this workflow

## Evidence

- [abstract] QCC/QCM quantification of cluster reconstruction: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA encoded space from validated data including immune signatures: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [discussion] Cluster 5 naïve T cell signature refinement: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [abstract] SCA selectivity of cluster reconstruction: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [readme] Docker-embedded reproducible SCA workflow: [evidence span withheld — anchor status: not_found]
