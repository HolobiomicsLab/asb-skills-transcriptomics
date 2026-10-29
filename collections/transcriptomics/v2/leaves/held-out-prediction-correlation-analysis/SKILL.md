---
name: held-out-prediction-correlation-analysis
description: 'Use when evaluating crossmodal generation quality of scMM (or similar
  multimodal VAEs) on CITE-seq data: need to quantify how precisely surface protein
  abundance can be predicted from transcriptome observations in held-out test cells
  via correlation.'
when_to_use_negative:
- The goal is joint representation/clustering of both modalities rather than quantitative
  prediction accuracy of one modality from the other
- Data lacks a paired second modality (e.g. transcriptome-only scRNA-seq), so no crossmodal
  ground truth exists for correlation
- Modalities are chromatin accessibility rather than surface protein counts and no
  modality-appropriate likelihood (e.g. scMM's ATAC-seq folder with peak.tsv setup)
  has been configured
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE used to learn joint representations and
    perform transcriptome-to-protein crossmodal generation with modality-specific
    count likelihoods
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Repository from which scMM's model implementation code was adopted
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Used for downstream analysis of scMM outputs (R/tutorial.R), including correlating
    predicted vs. original protein counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:2019028c015e74513f13d3f7ee4b63317a8f34ff65094fee3eb31101506289df
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# held-out-prediction-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a multimodal model's crossmodal generation by withholding one modality (surface proteins) from held-out test cells, generating predictions conditioned only on the transcriptome, and correlating predicted NB mean parameters against original counts.

## When to use

Use when evaluating crossmodal generation quality of scMM (or similar multimodal VAEs) on CITE-seq data: need to quantify how precisely surface protein abundance can be predicted from transcriptome observations in held-out test cells via correlation.

## When NOT to use

- The goal is joint representation/clustering of both modalities rather than quantitative prediction accuracy of one modality from the other
- Data lacks a paired second modality (e.g. transcriptome-only scRNA-seq), so no crossmodal ground truth exists for correlation
- Modalities are chromatin accessibility rather than surface protein counts and no modality-appropriate likelihood (e.g. scMM's ATAC-seq folder with peak.tsv setup) has been configured

## Inputs

- scRNA-seq transcriptome count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv feature/barcode files
- CITE-seq surface protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv
- Trained scMM mixture-of-experts multimodal VAE model

## Outputs

- nb_mean_protein_correlations.csv — per-protein Pearson correlations between estimated NB mean parameters and original surface protein counts on held-out test cells

## How to apply

Load the scRNA-seq transcriptome counts (gene.tsv, barcode.tsv) and CITE-seq protein modality files (Protein_count.mtx, protein.tsv, barcode.tsv) from the scMM data directory (data/BMNC-style folder layout with RNA-seq and CITE-seq subfolders). Split cells into training and held-out test sets, withholding the surface protein measurements of the test cells. Train the scMM mixture-of-experts multimodal variational autoencoder end-to-end on raw counts of both modalities using modality-specific probability distributions (codes adopted from the MMVAE repository). For each held-out test cell, condition the trained model on the transcriptome observation alone and perform crossmodal generation of the surface protein measurement, extracting the negative binomial mean parameters from the generated protein-count distribution as point estimates of protein abundance. Correlate the estimated NB mean parameters against the original surface protein counts for the held-out cells and write the results to nb_mean_protein_correlations.csv.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE used to learn joint representations and perform transcriptome-to-protein crossmodal generation with modality-specific count likelihoods) — https://github.com/kodaim1115/scMM
- **MMVAE** (Repository from which scMM's model implementation code was adopted) — https://github.com/iffsid/mmvae
- **R** (Used for downstream analysis of scMM outputs (R/tutorial.R), including correlating predicted vs. original protein counts)

## Examples

```
python train.py --data data/BMNC --modalities RNA-seq CITE-seq; Rscript R/tutorial.R  # compute nb_mean_protein_correlations.csv from held-out crossmodal predictions
```

## Evaluation signals

- Transcriptome-to-protein correlations between estimated NB mean parameters and original surface protein counts are high on held-out test cells, consistent with the precise crossmodal prediction reported in the scMM preprint (biorxiv 2021.02.18.431907)
- Correlation values are computed on cells whose protein measurements were strictly withheld during training (no leakage from test-cell protein counts)
- Output file nb_mean_protein_correlations.csv exists with one correlation entry per surface protein feature in protein.tsv
- Predicted values are negative binomial mean parameters extracted from the generated protein-count distribution, not sampled counts
- Barcodes in the output align with the barcode.tsv files of both the RNA-seq and CITE-seq modalities

## Limitations

- Evaluation on the scMM repository tutorial uses toy data — randomly subsampled 15,000 BMNC CITE-seq cells with only the 5000 most variable genes selected for transcriptome — so absolute correlations depend on the dataset and subsampling
- The MMVAE authors note recomputed analyses can yield different numeric values (relative performance against baselines is preserved), so exact correlation numbers may vary across runs
- Prediction accuracy depends on end-to-end training on raw counts with modality-specific likelihoods; retraining model implementations (from the MMVAE codebase) requires matching likelihood scaling between modalities

## Evidence
