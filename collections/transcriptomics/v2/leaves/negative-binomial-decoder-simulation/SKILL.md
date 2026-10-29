---
name: negative-binomial-decoder-simulation
description: Use when synthetic single-cell count data must be generated without relying
  on copies of the original cells — e.g., enlarging a small pilot scRNA-seq dataset
  (such as PBMC4k, 4182 cells × 1000 HVGs) after a VAE like scVI has been trained
  on it.
when_to_use_negative:
- When the goal is to faithfully reproduce the original pilot cells' clustering structure
  — the paper's scVI (posterior) variant is the standard approach for conditioning
  on observations, and posterior-vs-prior quality trade-offs should be compared rather
  than assuming the prior variant suffices.
- 'When batch effects must be modeled: the paper examined all models without batch
  correction capabilities and notes batch correction methods were outside its scope.'
- When the target technology is highly sparse 10x Genomics data with very small pilot
  sizes — the paper found inference from small to larger datasets more challenging
  for sparser 10x datasets than Smart-seq2.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scVI
  role: Trained VAE whose latent z (resampled from Normal(0,1)) and library sizes
    (resampled from Normal(l_mu,1)) are passed through the gamma-Poisson (negative
    binomial) decoder to generate synthetic UMI counts; paper used scvi 0.6.0 from
    bioconda.
  repo: https://github.com/scverse/scvi-tools
- name: scDBM-paper
  role: Reference implementation containing data preparation, Pluto notebooks, and
    analysis scripts for the scVI prior/posterior sampling variants.
  repo: https://github.com/MTreppner/scDBM-paper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:780dc379a3eb764bae58c50a1a1025aceaaeeafd4c17aa836f57798f29e72017
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - scDBM-paper
  - scVI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# negative-binomial-decoder-simulation

## Summary

A scVI-based generative variant for scRNA-seq that draws latent variables directly from the prior Normal(0,1) instead of the posterior p(z|x), and library sizes from Normal(l_mu, 1), then passes them through the gamma-Poisson (negative binomial) decoder to synthesize UMI counts. It serves as a fully generative alternative to the posterior-sampling variant, which still relies on copies of original observations.

## When to use

Use when synthetic single-cell count data must be generated without relying on copies of the original cells — e.g., enlarging a small pilot scRNA-seq dataset (such as PBMC4k, 4182 cells × 1000 HVGs) after a VAE like scVI has been trained on it.

## When NOT to use

- When the goal is to faithfully reproduce the original pilot cells' clustering structure — the paper's scVI (posterior) variant is the standard approach for conditioning on observations, and posterior-vs-prior quality trade-offs should be compared rather than assuming the prior variant suffices.
- When batch effects must be modeled: the paper examined all models without batch correction capabilities and notes batch correction methods were outside its scope.
- When the target technology is highly sparse 10x Genomics data with very small pilot sizes — the paper found inference from small to larger datasets more challenging for sparser 10x datasets than Smart-seq2.

## Inputs

- Preprocessed scRNA-seq count matrix of a pilot dataset (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Trained scVI model with learned latent encoder and library-size mean (l_mu)

## Outputs

- prior_sampling.py — script that demonstrably samples z from Normal(0,1) and library sizes from Normal(l_mu, 1)
- synthetic_counts_prior.csv — a cell-by-gene synthetic UMI count matrix produced by the gamma-Poisson decoder

## How to apply

Load the preprocessed pilot count matrix (e.g., PBMC4k with 4182 cells and 1000 highly variable genes) into the trained scVI model. In the Python scVI implementation (paper used scvi 0.6 installed via Conda from bioconda), replace the posterior sampling step p(z|x) with sampling latent z from Normal(0, 1), and replace the inferred library-size posterior with sampling library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean. Feed the prior-sampled z and library sizes through scVI's gamma-Poisson (negative binomial) decoder to obtain synthetic UMI counts. Save the sampling code (e.g., prior_sampling.py) and the generated expression matrix (e.g., synthetic_counts_prior.csv). The rationale, per the paper, is that the standard posterior variant conditions generation on original observations and showed high variability, while the prior variant generates de novo from the latent prior.

## Related tools

- **scVI** (Trained VAE whose latent z (resampled from Normal(0,1)) and library sizes (resampled from Normal(l_mu,1)) are passed through the gamma-Poisson (negative binomial) decoder to generate synthetic UMI counts; paper used scvi 0.6.0 from bioconda.) — https://github.com/scverse/scvi-tools
- **scDBM-paper** (Reference implementation containing data preparation, Pluto notebooks, and analysis scripts for the scVI prior/posterior sampling variants.) — https://github.com/MTreppner/scDBM-paper

## Examples

```
julia> using Conda; Conda.add("scvi=0.6", channel="bioconda")  # then in Python: z = Normal(0,1).sample(); l = Normal(l_mu,1).sample(); counts = decoder(z, l)
```

## Evaluation signals

- prior_sampling.py runs end-to-end without error.
- The code demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1) rather than from the posterior p(z|x).
- synthetic_counts_prior.csv is a cell-by-gene count matrix of the expected dimensionality (e.g., 4182 cells × 1000 genes for PBMC4k).
- Synthetic counts are non-negative integers consistent with gamma-Poisson (negative binomial) decoder output for UMI data.
- Downstream checks mirror the paper's evaluation: UMAP/Seurat clustering similarity (ARI, DBI), cluster proportions, and per-gene distribution statistics such as median, IQR, and proportion of zero counts.

## Limitations

- The paper's scVI (posterior) variant showed high variability in clustering similarity, likely due to amplifying artifacts of small datasets; prior-sampling results should be compared against this baseline rather than assumed superior.
- All approaches overrepresented highly abundant cell types and missed less abundant cell types, and had problems reproducing bimodal per-gene distributions.
- For sparser 10x Genomics datasets, inference from small to larger datasets was more challenging than for Smart-seq2 data.
- All models were examined without using potentially available batch correction capabilities.

## Evidence

- [abstract] scVI prior vs posterior variants: "First, we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution."
- [results] sampling from prior and posterior: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
- [readme] scvi installation version: "julia> using Conda
julia> Conda.add("scvi=0.6", channel="bioconda")"
