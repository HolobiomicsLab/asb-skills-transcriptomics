---
name: prior-latent-space-synthetic-generation
description: Use when generating a full-size synthetic scRNA-seq count matrix from
  a generative model trained on a small pilot sub-sample (e.g. 384–2304 cells), and
  you want to avoid amplifying pilot-specific artifacts that posterior sampling of
  p(z|x) introduces.
when_to_use_negative:
- The application requires batch-effect correction or the pilot data contain batch
  effects — the study examined all models without using potentially available batch
  correction capabilities, and scDBMs currently cannot learn batch effects
- Faithful reconstruction of cell-level identities from specific pilot cells is required
  — posterior sampling p(z|x) is the standard approach for this, whereas prior sampling
  deliberately decouples generated cells from individual pilot cells
- The target quantity is rare cell-type proportions in a very small pilot — all approaches
  overrepresent highly abundant cell types and miss less abundant ones, especially
  for sparse 10x Genomics data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3316
tools:
- name: scVI
  role: Variational autoencoder trained on pilot sub-samples; the prior variant samples
    latent z from Normal(0,1) and library sizes from Normal(l_mu, 1) and decodes them
    into synthetic counts
  repo: https://github.com/scverse/scvi-tools
- name: scDBM-paper
  role: Analysis code (Pluto notebooks, data preparation Rmd scripts) that installs
    scvi 0.6 via Conda and runs the prior- and posterior-sampling experiments
  repo: https://github.com/MTreppner/scDBM-paper
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:e625ecb98194ff990c4de46987f9a129cbcaaff1a45f84c0567bf7439166e21b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM-paper
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# prior-latent-space-synthetic-generation

## Summary

A scVI sampling variant that generates synthetic scRNA-seq cells by drawing latent z from the prior Normal(0,1) and library sizes from Normal(l_mu, 1) instead of encoding pilot cells to the posterior p(z|x). It was introduced because posterior sampling amplifies artifacts of small pilot datasets.

## When to use

Use when generating a full-size synthetic scRNA-seq count matrix from a generative model trained on a small pilot sub-sample (e.g. 384–2304 cells), and you want to avoid amplifying pilot-specific artifacts that posterior sampling of p(z|x) introduces.

## When NOT to use

- The application requires batch-effect correction or the pilot data contain batch effects — the study examined all models without using potentially available batch correction capabilities, and scDBMs currently cannot learn batch effects
- Faithful reconstruction of cell-level identities from specific pilot cells is required — posterior sampling p(z|x) is the standard approach for this, whereas prior sampling deliberately decouples generated cells from individual pilot cells
- The target quantity is rare cell-type proportions in a very small pilot — all approaches overrepresent highly abundant cell types and miss less abundant ones, especially for sparse 10x Genomics data

## Inputs

- Preprocessed count matrix of a small pilot sub-sample (e.g. PBMC4k sub-samples of 384–2304 cells over 1000 highly variable genes, or Segerstolpe sub-samples over 2000 HVGs)
- Trained scVI model (adapted Python implementation, scvi 0.6.0, 1 hidden layer, latent dimension 10)
- Mean library size parameter l_mu estimated from the pilot data

## Outputs

- Synthetic count matrix with the original study's cell count (e.g. 4182 PBMC4k cells), one per (dataset x pilot size x repetition)
- Cluster labels for synthetic cells obtained by nearest-neighbor mapping to original data in UMAP space
- Evaluation metrics: adjusted Rand index, Davies–Bouldin index, cluster proportion differences, per-gene MADs of medians/IQR/CV/zero proportions

## How to apply

Train the adapted Python scVI implementation (scvi 0.6.0 installed via Conda from bioconda) on a random sub-sampled pilot dataset with the default architecture: one hidden layer for encoder and decoder, latent space dimensionality 10. Instead of replicating pilot cells and sampling from the posterior p(z|x), draw latent variables z from Normal(0, 1) and library sizes from Normal(l_mu, 1), then decode them through the trained generative network. Stack decoded samples until the synthetic dataset reaches the original study size (e.g. 4182 cells for PBMC4k, 2090 for Segerstolpe). The rationale is that the posterior variant showed high variability in clustering similarity (adjusted Rand index), most likely due to amplifying artifacts of small datasets, whereas prior sampling shows much lower variability across 30 repetitions per pilot size. Validate the synthetic data by mapping synthetic observations to original data in UMAP space and computing DBI, ARI, cluster proportions, and per-gene MAD statistics.

## Related tools

- **scVI** (Variational autoencoder trained on pilot sub-samples; the prior variant samples latent z from Normal(0,1) and library sizes from Normal(l_mu, 1) and decodes them into synthetic counts) — https://github.com/scverse/scvi-tools
- **scDBM-paper** (Analysis code (Pluto notebooks, data preparation Rmd scripts) that installs scvi 0.6 via Conda and runs the prior- and posterior-sampling experiments) — https://github.com/MTreppner/scDBM-paper

## Examples

```
julia> using Conda; Conda.add("scvi=0.6", channel="bioconda")
```

## Evaluation signals

- For every combination of dataset (2) x model (3) x pilot size (6) x repetition (30), a synthetic count matrix with the original study's cell count was produced — 1080 generated artifacts in total
- Adjusted Rand index of clustering similarity between synthetic and ground-truth labels shows markedly lower variability for prior sampling than for the posterior variant
- Cluster proportions of synthetic cells approach ground-truth proportions as pilot size increases from 384 to 2304 cells
- Per-gene mean absolute deviations (medians, IQRs, coefficients of variation, zero proportions) are comparable to or better than the negative binomial noise baseline with scale ~ Uniform(0,5) and shape ~ Uniform(0,10)

## Limitations

- The scVI (posterior) variant showed high variability attributed to amplifying artifacts of small datasets — the motivating failure mode this prior-sampling variant avoids, but all variants still overrepresent abundant and miss rare cell types
- All approaches learn the univariate distribution of most genes but fail on bimodal gene distributions
- Inference from small to larger datasets is more challenging for sparser 10x Genomics datasets (PBMC4k) than Smart-seq2 (Segerstolpe)
- Downstream label evaluation relies on the unverified assumption that synthetic cells mapped to the original data are mapped to the correct clusters

## Evidence

- [abstract] the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets: "we find that the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets."
- [results] scVI prior vs posterior sampling definition: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
- [methods] prior sampling variant definition from workflow: [evidence span withheld — anchor status: not_found]
- [readme] scVI installation and version in paper repo: [evidence span withheld — anchor status: external_doc]
