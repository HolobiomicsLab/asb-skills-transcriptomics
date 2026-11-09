---
name: transcription-factor-marker-prioritization
description: Use when scRNAseq clusters have been defined (e.g., via rCASC clustering)
  and the goal is to identify transcription factors whose target-derived latent representation
  drives cluster aggregation, rather than generic differential expression markers.
when_to_use_negative:
- The goal is generic marker-gene discovery without a functional/TF interpretation
  — plain differential expression suffices and SCA's TF-target space is unnecessary.
- 'miRNAs must be quantified alongside mRNAs to build TF-miRNA circuits: the article
  notes it is not currently possible to quantify miRNAs at single-cell level together
  with mRNAs.'
- The dataset is already a feature table / latent embedding — re-encoding with SCA
  would add complexity without benefit (the vSCA variant showed no specific improvement
  over plain TF-based SCA).
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCA
  role: Sparsely-Connected Autoencoder that builds a TF-target-derived encoded space
    used to rank cluster-specific transcription factors
  repo: github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework in which SCA is implemented as a module (with GUI), providing the
    autoencoderAnalysis4Clustering wrappers and reproducible Docker-based execution
  repo: github.com/kendomaniac/rCASC
- name: Docker
  role: Daemon hosting the repbioinfo containers that guarantee computational reproducibility
    of the SCA/rCASC workflow
- name: R
  role: Environment in which rCASC (installed via devtools install_github) is run
- name: devtools
  role: 'Used to install rCASC from GitHub: install_github("kendomaniac/rCASC")'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:bc868c6eb448789e0b8c34c0fe142f92f73ba091871dab206f3f4593dca2a7ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - R
  - SCA
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transcription-factor-marker-prioritization

## Summary

Ranks transcription factors as functional markers of single-cell clusters using the SCA (Sparsely-Connected Autoencoder) encoded space built from TF-target priors, validated against known biology (e.g., CHD4 for B cells, CEBPA for monocytes in the setA dataset).

## When to use

Use when scRNAseq clusters have been defined (e.g., via rCASC clustering) and the goal is to identify transcription factors whose target-derived latent representation drives cluster aggregation, rather than generic differential expression markers.

## When NOT to use

- The goal is generic marker-gene discovery without a functional/TF interpretation — plain differential expression suffices and SCA's TF-target space is unnecessary.
- miRNAs must be quantified alongside mRNAs to build TF-miRNA circuits: the article notes it is not currently possible to quantify miRNAs at single-cell level together with mRNAs.
- The dataset is already a feature table / latent embedding — re-encoding with SCA would add complexity without benefit (the vSCA variant showed no specific improvement over plain TF-based SCA).

## Inputs

- Filtered single-cell UMI counts table (e.g., from cellrangerCount or indropCounts, optionally preprocessed with filterZeros/lorenzFilter/topx)
- Cluster 1/2 (or other) cell-annotation files from rCASC clustering
- Experimentally validated TF-target prior used to define the SCA latent space

## Outputs

- Ranked list of transcription factors per cluster (e.g., COMET output: CHD4 for cluster 1, CEBPA for cluster 2 of setA)
- QCM and QCC trajectories quantifying robustness of the SCA encoding per cluster
- SCA signatures (TF-based cluster signatures, e.g., the 4-gene signatures for setA clusters 1 and 2)

## How to apply

Load the filtered single-cell counts table and cluster annotations into the R/rCASC environment executed through the Docker daemon. Build the sparsely-connected autoencoder latent space from an experimentally validated TF-target prior (alternatives: miRNA targets, kinase targets, cancer-related immune signatures), so that hidden-layer weights encode TF activity rather than individual noisy genes. Run the SCA permutation-based stability assessment via the autoencoderAnalysis4Clustering wrappers and compute QCM (Quality Control of Model) and QCC (Quality Control of Cluster) trajectories to confirm the encoded space robustly reconstructs the target cluster. Rank cluster-specific TFs from the encoded space — e.g., using COMET (cometsc/cometsc2) — and interpret top-ranked TFs against Table 1 of the paper: cluster 1 of setA (B cells) is marked by CHD4 and cluster 2 (monocytes) by CEBPA. Since TFs are lowly expressed and dropout-prone, the TF-target latent space, not raw expression, is the basis of ranking.

## Related tools

- **SCA** (Sparsely-Connected Autoencoder that builds a TF-target-derived encoded space used to rank cluster-specific transcription factors) — github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework in which SCA is implemented as a module (with GUI), providing the autoencoderAnalysis4Clustering wrappers and reproducible Docker-based execution) — github.com/kendomaniac/rCASC
- **Docker** (Daemon hosting the repbioinfo containers that guarantee computational reproducibility of the SCA/rCASC workflow)
- **R** (Environment in which rCASC (installed via devtools install_github) is run)
- **devtools** (Used to install rCASC from GitHub: install_github("kendomaniac/rCASC"))

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")  # then run SCA/COMET via autoencoderAnalysis4Clustering on setA cluster 1/2 annotations
```

## Evaluation signals

- QCM and QCC trajectories confirm the SCA encoding is stable and the target cluster is among those reconstructed by the encoded space
- Top-ranked TF for each cluster matches biologically validated markers: CHD4 for setA cluster 1 (B cells) and CEBPA for cluster 2 (monocytes), as in Table 1
- The identified TFs are members of the reported 4-gene SCA signatures for the respective clusters
- TFs are ranked from the SCA encoded (TF-target) space rather than raw counts, consistent with dropout affecting lowly expressed TF genes

## Limitations

- Transcription factors and kinases are lowly expressed and notably affected by dropout in scRNA-seq, biasing raw-expression-based ranking.
- scRNAseq is very noisy and produces more zeros than expected, with greater bias for poorly expressed genes.
- SCA reconstructs only specific clusters — it indicates only those clusters where the encoding space is key to cell aggregation; clusters not reconstructed yield no TF signature.
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured.

## Evidence

- [abstract] SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [discussion] top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes.: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] Transcription factors and kinases are encoded by genes characterized by a relatively low expression in cells, thus they can be notably affected by dropout.: "Transcription factors and kinases are encoded by genes characterized by a relatively low expression in cells, thus they can be notably affected by dropout."
- [abstract] we implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [methods] The following commands allow the rCASC installation in [R] environment: "The following commands allow the rCASC installation in [*R*](https://cran.r-project.org/) environment"
