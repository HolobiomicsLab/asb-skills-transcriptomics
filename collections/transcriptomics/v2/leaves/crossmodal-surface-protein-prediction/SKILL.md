---
name: crossmodal-surface-protein-prediction
description: Use when CITE-seq transcriptome counts are available for a new batch
  (e.g. BMNC) and you must predict surface protein levels for proteins shared with
  a pretrained scMM model, then compare against a Seurat baseline.
when_to_use_negative:
- The target batch's protein panel shares no (or too few) proteins with the pretrained
  scMM model's 24-protein training panel, since crossmodal prediction is restricted
  to shared features
- Measured protein counts are already available and the goal is joint multimodal clustering
  or interpretability (pseudocells, latent regulatory programs) rather than crossmodal
  imputation
- The data are not single-cell count matrices in the scMM CITE-seq folder layout (e.g.
  already-normalized feature tables)
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMM
  role: Pretrained mixture-of-experts multimodal VAE used to generate crossmodal surface
    protein predictions from transcriptome-only input
  repo: https://github.com/kodaim1115/scMM
- name: Seurat
  role: Anchor-based transfer baseline whose predicted surface protein levels are
    compared to scMM via per-protein Jaccard index
- name: R
  role: Used for downstream analysis of scMM outputs and computation of Jaccard index
    scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:8f0674987862ef5812c3c494d48a61542129612e735622c7a0543ac55f406bf5
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
  - R
  - Seurat
  - scMM
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# crossmodal-surface-protein-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts surface protein abundance from transcriptome-only CITE-seq input using the pretrained scMM mixture-of-experts multimodal VAE, and benchmarks prediction quality against a Seurat anchor-based transfer baseline using per-protein Jaccard index (JI).

## When to use

Use when CITE-seq transcriptome counts are available for a new batch (e.g. BMNC) and you must predict surface protein levels for proteins shared with a pretrained scMM model, then compare against a Seurat baseline.

## When NOT to use

- The target batch's protein panel shares no (or too few) proteins with the pretrained scMM model's 24-protein training panel, since crossmodal prediction is restricted to shared features
- Measured protein counts are already available and the goal is joint multimodal clustering or interpretability (pseudocells, latent regulatory programs) rather than crossmodal imputation
- The data are not single-cell count matrices in the scMM CITE-seq folder layout (e.g. already-normalized feature tables)

## Inputs

- BMNC CITE-seq protein count matrix (Protein_count.mtx)
- Gene feature file (gene.tsv)
- Protein feature file (protein.tsv)
- Single-cell barcode file (barcode.tsv)
- BMNC transcriptome count matrix (RNA_count.mtx)
- Pretrained scMM mixture-of-experts multimodal VAE model trained on PBMC CITE-seq data
- Seurat R package for the anchor-based transfer baseline

## Outputs

- Predicted abundance values for the 24 shared surface proteins for all ~30,000 BMNC cells
- Per-protein Jaccard index scores for scMM and the Seurat baseline, aggregated in crossmodal_prediction_ji.csv

## How to apply

Load the target CITE-seq matrices (Protein_count.mtx, gene.tsv, barcode.tsv, protein.tsv) from the scMM data directory following the README folder layout (RNA-seq/ and CITE-seq/ subfolders). Apply the pretrained PBMC scMM mixture-of-experts multimodal VAE in crossmodal generation mode to predict the 24 shared surface proteins for all ~30,000 BMNC cells from transcriptome-only input. Run the Seurat anchor-based transfer baseline on the same BMNC data to predict the identical 24 proteins. For each protein and each method, compute the Jaccard index (JI) between the predicted and measured top-expressing cell sets, then aggregate per-protein JI scores for scMM and Seurat into crossmodal_prediction_ji.csv. Correct application is confirmed when scMM's aggregated JI exceeds the state-of-the-art Seurat baseline, as reported in the paper.

## Related tools

- **scMM** (Pretrained mixture-of-experts multimodal VAE used to generate crossmodal surface protein predictions from transcriptome-only input) — https://github.com/kodaim1115/scMM
- **Seurat** (Anchor-based transfer baseline whose predicted surface protein levels are compared to scMM via per-protein Jaccard index)
- **R** (Used for downstream analysis of scMM outputs and computation of Jaccard index scores)

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Per-protein JI is computed between predicted and measured top-expressing cell sets for each of the 24 shared surface proteins
- Aggregated JI for scMM exceeds the Seurat anchor-based transfer baseline, reproducing the reported superiority
- Predictions cover all ~30,000 BMNC cells from transcriptome-only input (no protein counts fed to the model)
- crossmodal_prediction_ji.csv contains paired per-protein JI rows for both scMM and Seurat methods
- Input files conform to the scMM folder layout (RNA-seq/RNA_count.mtx, CITE-seq/Protein_count.mtx with gene.tsv, protein.tsv, barcode.tsv)

## Limitations

- The pretrained model was trained on PBMC data, so only the 24 proteins shared between PBMC and BMNC panels can be predicted on the target batch
- The distributed tutorial toy data are randomly subsampled (15,000 BMNC cells, top 5,000 most variable genes), so reproductions on the full ~30,000-cell batch may differ numerically from the toy workflow
- Crossmodal prediction accuracy depends on transcriptome-protein coupling; the paper's claim is relative superiority over state-of-the-art and conventional approaches, not absolute prediction accuracy

## Evidence
