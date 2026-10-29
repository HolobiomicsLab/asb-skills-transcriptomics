---
name: negative-binomial-mean-extraction
description: Use when scMM has been trained on transcriptome + surface-protein CITE-seq
  counts and you must score held-out transcriptome-to-protein crossmodal generation
  via NB mean-parameter correlations.
when_to_use_negative:
- Input modalities are not raw counts (e.g. normalized or log-transformed expression),
  since scMM models raw count data with modality-specific likelihoods including the
  negative binomial for proteins
- The target modality is chromatin accessibility rather than surface proteins, where
  the count distribution and feature files (peak.tsv, ATAC-seq folder) differ from
  the CITE-seq protein setup
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMM
  role: Deep generative framework performing the crossmodal generation whose NB mean
    parameters are extracted as protein point estimates
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Repository from which scMM's mixture-of-experts multimodal VAE code was adopted
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Used for downstream analysis of scMM outputs via R/tutorial.R, e.g. correlating
    NB mean estimates against original counts
  repo: https://github.com/kodaim1115/scMM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:17986dfc8f36c9770904c3696e7d203f4cf50fb19316c075e1261e78b57ae554
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kodaim1115/scMM
  repo_url: https://github.com/kodaim1115/scMM
  tools:
  - MMVAE
  - R
  - scMM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# negative-binomial-mean-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the negative binomial (NB) mean parameters from scMM's crossmodally generated protein-count distributions as point estimates of surface protein abundance, then validates them by correlating against original counts for held-out CITE-seq cells.

## When to use

Use when scMM has been trained on transcriptome + surface-protein CITE-seq counts and you must score held-out transcriptome-to-protein crossmodal generation via NB mean-parameter correlations.

## When NOT to use

- Input modalities are not raw counts (e.g. normalized or log-transformed expression), since scMM models raw count data with modality-specific likelihoods including the negative binomial for proteins
- The target modality is chromatin accessibility rather than surface proteins, where the count distribution and feature files (peak.tsv, ATAC-seq folder) differ from the CITE-seq protein setup

## Inputs

- scRNA-seq transcriptome counts (gene.tsv, barcode.tsv) from the scMM RNA-seq data folder
- CITE-seq surface protein counts (Protein_count.mtx, protein.tsv, barcode.tsv) from the scMM CITE-seq data folder
- Trained scMM mixture-of-experts multimodal VAE model (PyTorch, MMVAE-based)
- Train/test cell split with protein measurements withheld for test cells

## Outputs

- Per-protein NB mean parameter point estimates of surface protein abundance for held-out cells
- nb_mean_protein_correlations.csv containing correlations between estimated NB means and original protein counts

## How to apply

After training the scMM mixture-of-experts multimodal VAE (code adopted from the MMVAE repository) end-to-end on raw RNA and protein counts with modality-specific likelihoods, withhold the protein measurements of test cells and condition the trained model on their transcriptome alone to perform crossmodal generation. Because the protein modality is modeled with a negative binomial likelihood on raw counts, take the NB mean parameters of the generated protein-count distribution as point estimates of protein abundance. Finally, correlate these estimated NB means against the original surface protein counts for the held-out cells (e.g. per protein) and write the per-protein correlations to nb_mean_protein_correlations.csv. Rationale: scMM models each modality's raw counts with different probability distributions, so the NB mean is the natural, distribution-aware point estimate rather than a raw sampled count.

## Related tools

- **scMM** (Deep generative framework performing the crossmodal generation whose NB mean parameters are extracted as protein point estimates) — https://github.com/kodaim1115/scMM
- **MMVAE** (Repository from which scMM's mixture-of-experts multimodal VAE code was adopted) — https://github.com/iffsid/mmvae
- **R** (Used for downstream analysis of scMM outputs via R/tutorial.R, e.g. correlating NB mean estimates against original counts) — https://github.com/kodaim1115/scMM

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Transcriptome-to-protein correlations between estimated NB mean parameters and original surface protein counts are high on held-out test data
- Result is consistent with the precise crossmodal prediction reported in the scMM preprint (biorxiv 2021.02.18.431907)
- Output file nb_mean_protein_correlations.csv is produced with one correlation per protein and correct barcodes/feature alignment between barcode.tsv and protein.tsv
- Generated protein distributions are proper NB distributions (valid mean parameters) rather than raw sampled counts

## Limitations

- Performance depends on training on raw counts with modality-specific likelihoods; scMM's crossmodal generation advantage was demonstrated relative to state-of-the-art and conventional approaches, so absolute correlation levels may vary by dataset
- The tutorial data is a randomly subsampled 15,000-cell BMNC CITE-seq dataset with 5,000 most variable genes; results on other tissues or preprocessing choices are not guaranteed by the README

## Evidence
