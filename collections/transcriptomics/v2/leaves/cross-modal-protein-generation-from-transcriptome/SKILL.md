---
name: cross-modal-protein-generation-from-transcriptome
description: Use when you have paired CITE-seq transcriptome and surface protein counts
  (RNA_count.mtx/gene.tsv and Protein_count.mtx/protein.tsv with barcode.tsv) and
  need to predict protein measurements for cells whose protein data is withheld, given
  transcriptome observations alone.
when_to_use_negative:
- Data is not paired CITE-seq transcriptome-plus-protein counts (e.g. chromatin accessibility
  data, which scMM handles under ATAC-seq with peak.tsv, requiring a different crossmodal
  setup)
- You need joint latent representations or clustering/interpretability (pseudocell-based
  multimodal regulatory programs) rather than one-directional protein prediction from
  transcriptome
- Only unpaired single-cell transcriptome and protein measurements exist, since the
  model requires end-to-end training on raw counts of both modalities from the same
  cells
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMM
  role: Mixture-of-experts multimodal variational autoencoder trained end-to-end on
    raw transcriptome and protein counts; conditions on transcriptome to cross-modally
    generate protein counts with negative binomial likelihoods
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Source repository from which the scMM mixture-of-experts VAE model code was
    adopted
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Used for downstream analysis of scMM outputs (R/tutorial.R), including correlation
    of estimated protein abundances against measured counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:370eaa1df379e89d1a509c73784bef8695d7b233aa201ddd6883f094421f4193
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cross-modal-protein-generation-from-transcriptome

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts CITE-seq surface protein abundances from transcriptome-only observations using scMM's mixture-of-experts multimodal variational autoencoder, extracting negative binomial mean parameters as point estimates and validating them by correlation against held-out protein counts.

## When to use

Use when you have paired CITE-seq transcriptome and surface protein counts (RNA_count.mtx/gene.tsv and Protein_count.mtx/protein.tsv with barcode.tsv) and need to predict protein measurements for cells whose protein data is withheld, given transcriptome observations alone.

## When NOT to use

- Data is not paired CITE-seq transcriptome-plus-protein counts (e.g. chromatin accessibility data, which scMM handles under ATAC-seq with peak.tsv, requiring a different crossmodal setup)
- You need joint latent representations or clustering/interpretability (pseudocell-based multimodal regulatory programs) rather than one-directional protein prediction from transcriptome
- Only unpaired single-cell transcriptome and protein measurements exist, since the model requires end-to-end training on raw counts of both modalities from the same cells

## Inputs

- scRNA-seq transcriptome count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv
- CITE-seq surface protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv
- Training/test cell split specification (protein counts withheld for test cells)

## Outputs

- Per-protein negative binomial mean parameter estimates for held-out cells (point estimates of surface protein abundance)
- Correlations between estimated NB mean parameters and original protein counts (nb_mean_protein_correlations.csv)

## How to apply

Load the scRNA-seq transcriptome counts (gene.tsv, barcode.tsv) and the CITE-seq protein modality files (Protein_count.mtx, protein.tsv, barcode.tsv) from the scMM data directory, then split cells into training and held-out test sets, withholding the surface protein measurements of the test cells. Train the scMM mixture-of-experts multimodal variational autoencoder end-to-end on raw counts of both modalities; the model uses modality-specific likelihoods (negative binomial for count data) and code adopted from the MMVAE repository. For each held-out test cell, condition the trained model on the transcriptome observation alone and perform cross-modal generation of the surface protein measurement. Extract the negative binomial mean parameters from the generated protein-count distribution as point estimates of protein abundance. Finally, correlate the estimated NB mean parameters against the original surface protein counts for the held-out cells and write the results to nb_mean_protein_correlations.csv.

## Related tools

- **scMM** (Mixture-of-experts multimodal variational autoencoder trained end-to-end on raw transcriptome and protein counts; conditions on transcriptome to cross-modally generate protein counts with negative binomial likelihoods) — https://github.com/kodaim1115/scMM
- **MMVAE** (Source repository from which the scMM mixture-of-experts VAE model code was adopted) — https://github.com/iffsid/mmvae
- **R** (Used for downstream analysis of scMM outputs (R/tutorial.R), including correlation of estimated protein abundances against measured counts)

## Examples

```
Rscript R/tutorial.R  # downstream analysis correlating scMM crossmodally generated protein NB mean parameters against measured CITE-seq protein counts
```

## Evaluation signals

- Correlations between estimated NB mean parameters and original surface protein counts on held-out test cells are high, matching the precise crossmodal prediction reported in the scMM preprint (biorxiv 2021.02.18.431907)
- nb_mean_protein_correlations.csv is produced with per-protein correlation values
- Estimated NB mean parameters fall within the count scale of the original protein measurements (non-negative, comparable magnitude to measured counts)
- Prediction quality exceeds state-of-the-art and conventional crossmodal prediction/integration baselines as claimed for scMM

## Limitations

- Reported performance is demonstrated on CITE-seq datasets (PBMC and toy BMNC data with randomly subsampled 15,000 cells and top 5,000 variable genes); behavior on other tissue types or modalities must be re-validated
- The MMVAE-derived model code base was tested on older dependencies (e.g. python 3.6.8, torch 1.3.1), so reproducibility may require a matching environment
- Training requires paired raw counts of both modalities from the same cells; protein measurements must be masked only at prediction time, not missing during training

## Evidence

- [intro] scMM is a novel deep generative model-based framework for the extraction of interpretable joint representations and cross-modal generation for single-cell multiomics data: "scMM is a novel deep generative model-based framework for the extraction of interpretable joint representations and cross-modal generation for single-cell multiomics data"
- [intro] achieves end-to-end learning by modeling raw count data in each modality based on different probability distributions: "achieves end-to-end learning by modeling raw count data in each modality based on different probability distributions"
