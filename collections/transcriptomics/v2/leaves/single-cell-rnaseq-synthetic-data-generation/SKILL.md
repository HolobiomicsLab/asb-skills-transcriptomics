---
name: single-cell-rnaseq-synthetic-data-generation
description: Use when a small pilot scRNA-seq count matrix is available and larger
  synthetic data of the original study size is needed to plan downstream analyses
  such as clustering or cell-type abundance estimation.
when_to_use_negative:
- When the goal is batch correction or batch-effect-aware integration — the paper
  examined all models without batch correction capabilities and scDBMs currently cannot
  learn batch effects
- When the pilot sample is not representative of the target experiment (experimental
  settings changed), since conclusions from small to larger datasets require representativeness
- When the target dataset is highly sparse (10x Genomics) and the pilot is very small
  — inference from small to larger datasets is more challenging for sparser 10x data
  than Smart-seq2
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scVI
  role: Variational autoencoder trained on pilot sub-samples; used in two generative
    variants sampling from the posterior or from the prior (z ~ Normal(0,1), library
    size ~ Normal(l_mu, 1)) with a gamma-Poisson decoder
  repo: https://github.com/scverse/scvi-tools
- name: scDBM
  role: Single-cell deep Boltzmann machine trained on pilot sub-samples; synthetic
    observations generated via Gibbs sampling
  repo: https://github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Julia implementation underlying scDBM training and Gibbs sampling
  repo: https://github.com/MTreppner/scDBM.jl
- name: UMAP
  role: Dimensionality reduction used to map synthetic observations to original data
    and acquire cluster labels
- name: Seurat
  role: Clustering of original data providing reference clusters and the reference
    Davies–Bouldin index
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:ff5c0667b941f42a2dd398f52787a66a072cb06dc1aa449e4550ac7397505f63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - BoltzmannMachines.jl
  - Seurat
  - UMAP
  - scDBM
  - scVI
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-rnaseq-synthetic-data-generation

## Summary

Generates synthetic scRNA-seq count matrices by training deep generative models (scVI in posterior- and prior-sampling variants, scDBM via Gibbs sampling) on small pilot datasets (e.g. PBMC4k, 4182 cells, 1000 highly variable genes), enabling inference from pilot experiments to full-scale study designs.

## When to use

Use when a small pilot scRNA-seq count matrix is available and larger synthetic data of the original study size is needed to plan downstream analyses such as clustering or cell-type abundance estimation.

## When NOT to use

- When the goal is batch correction or batch-effect-aware integration — the paper examined all models without batch correction capabilities and scDBMs currently cannot learn batch effects
- When the pilot sample is not representative of the target experiment (experimental settings changed), since conclusions from small to larger datasets require representativeness
- When the target dataset is highly sparse (10x Genomics) and the pilot is very small — inference from small to larger datasets is more challenging for sparser 10x data than Smart-seq2

## Inputs

- Preprocessed PBMC4k count matrix (4182 cells, 1000 highly variable genes)
- Preprocessed Segerstolpe count matrix (Smart-seq2 protocol)
- Sub-sampled pilot datasets of 384, 768, 1152, 1536, 1920, or 2304 cells
- Gene names and cluster label files (e.g. segerstolpe_hvg_clustering.csv)

## Outputs

- Synthetic cell-by-gene UMI count matrix (e.g. synthetic_counts_prior.csv)
- Trained scVI / scDBM generative models per pilot sub-sample
- Prior-sampling generation code (e.g. prior_sampling.py)

## How to apply

First sub-sample the original dataset (e.g. PBMC4k or Segerstolpe) into pilot datasets of 384–2304 cells by drawing each cell with equal probability. Train scVI (Python, scvi=0.6 via Conda/bioconda) or scDBM (Julia 1.5.0, BoltzmannMachines.jl) on each pilot sub-sample, then generate synthetic data at the size of the original study. For scVI, use two generative variants: the standard posterior sampling p(z|x), or the prior variant that draws latent z from Normal(0, 1) and library sizes from Normal(l_mu, 1) with l_mu the learned library-size mean, feeding both through scVI's gamma-Poisson (negative binomial) decoder to produce UMI counts. For scDBM, sample via Gibbs sampling. Validate outputs by mapping synthetic observations to original data via Euclidean distance in UMAP space and evaluating with the Davies–Bouldin index, adjusted Rand index, cluster proportions, and per-gene distributions (medians, IQR, coefficient of variation, zero-count proportions).

## Related tools

- **scVI** (Variational autoencoder trained on pilot sub-samples; used in two generative variants sampling from the posterior or from the prior (z ~ Normal(0,1), library size ~ Normal(l_mu, 1)) with a gamma-Poisson decoder) — https://github.com/scverse/scvi-tools
- **scDBM** (Single-cell deep Boltzmann machine trained on pilot sub-samples; synthetic observations generated via Gibbs sampling) — https://github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Julia implementation underlying scDBM training and Gibbs sampling) — https://github.com/MTreppner/scDBM.jl
- **UMAP** (Dimensionality reduction used to map synthetic observations to original data and acquire cluster labels)
- **Seurat** (Clustering of original data providing reference clusters and the reference Davies–Bouldin index)

## Examples

```
julia> using Conda; Conda.add("scvi=0.6", channel="bioconda")
```

## Evaluation signals

- prior_sampling.py runs end-to-end without error and demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1) rather than from the posterior
- synthetic_counts_prior.csv is a cell-by-gene count matrix of the expected dimensionality (matching the 1000 HVG gene space)
- Adjusted Rand index of clustering on synthetic data compared against ground-truth clustering of the original data
- Cluster proportions of synthetic cells converge toward ground-truth proportions as pilot dataset size increases (384 → 2304 cells)
- Mean absolute deviation per gene for medians, interquartile ranges, coefficients of variation, and proportion of zero counts compared between original and synthetic data; expect good recovery of univariate distributions but errors on bimodal genes

## Limitations

- The scVI (posterior) variant showed high variability in clustering similarity, likely due to amplifying artifacts of small datasets
- All approaches overrepresented highly abundant cell types and missed less abundant cell types
- All approaches learn the univariate distribution of most genes but had problems with bimodality
- For sparser 10x Genomics datasets it is more challenging to make an inference from small to larger datasets than for Smart-seq2
- Cluster-label assignment relies on the unverified assumption that synthetic cells mapped to the original data land in the correct clusters; batch correction capabilities were not used

## Evidence

- [abstract] we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution.: "First, we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution."
- [results] We took 30 sub-samples of 384, 768, 1152, 1536, 1920, and 2304 cells of the original dataset, trained the scDBM and scVI on these sub-samples: "We took 30 sub-samples of 384, 768, 1152, 1536, 1920, and 2304 cells of the original dataset, trained the scDBM and scVI on these sub-samples"
- [results] we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
- [results] acquired the cluster labels by mapping the synthetic observations to the original data based on the Euclidean distance in UMAP space: "acquired the cluster labels by mapping the synthetic observations to the original data based on the Euclidean distance in UMAP space"
- [readme] To run the notebooks... scVI must be installed... Conda.add("scvi=0.6", channel="bioconda"): "julia> using Conda
julia> Conda.add("scvi=0.6", channel="bioconda")"
