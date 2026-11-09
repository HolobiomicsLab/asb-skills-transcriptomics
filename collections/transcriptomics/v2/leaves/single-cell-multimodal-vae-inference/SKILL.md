---
name: single-cell-multimodal-vae-inference
description: Use when you have single-cell transcriptome count data (RNA_count.mtx
  with gene.tsv/barcode.tsv) from a CITE-seq experiment and need to predict paired
  surface protein abundances from a pretrained multimodal VAE, then quantify prediction
  accuracy against measured protein counts.
when_to_use_negative:
- Input is single-modality data with no measured protein counts available to validate
  predictions against (Jaccard index benchmarking requires measured top-expressing
  cell sets)
- You need to train a model from scratch rather than apply a pretrained one — use
  the scMM training workflow with paired multimodal input instead
- Data are chromatin accessibility rather than CITE-seq transcriptome/protein pairs
  (would require the ATAC-seq folder layout and peak.tsv features)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3474
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE used to generate crossmodal predictions
    of surface proteins from transcriptome-only input
  repo: https://github.com/kodaim1115/scMM
- name: Seurat
  role: Anchor-based transfer baseline for cross-modal surface protein prediction,
    used for Jaccard index comparison against scMM
- name: R
  role: Environment for downstream analysis of scMM outputs (R/tutorial.R) and for
    the Seurat baseline and Jaccard index computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:f27f24c04af5c84e34361187aea593c84360cefa0815f268db19d354aa4112ac
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-multimodal-vae-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the pretrained scMM mixture-of-experts multimodal VAE to perform crossmodal generation on CITE-seq data — e.g. predicting surface protein abundance of BMNC cells from transcriptome-only input — and benchmarks predictions against measured values using the Jaccard index.

## When to use

Use when you have single-cell transcriptome count data (RNA_count.mtx with gene.tsv/barcode.tsv) from a CITE-seq experiment and need to predict paired surface protein abundances from a pretrained multimodal VAE, then quantify prediction accuracy against measured protein counts.

## When NOT to use

- Input is single-modality data with no measured protein counts available to validate predictions against (Jaccard index benchmarking requires measured top-expressing cell sets)
- You need to train a model from scratch rather than apply a pretrained one — use the scMM training workflow with paired multimodal input instead
- Data are chromatin accessibility rather than CITE-seq transcriptome/protein pairs (would require the ATAC-seq folder layout and peak.tsv features)

## Inputs

- BMNC CITE-seq protein count matrix (Protein_count.mtx)
- Gene feature file (gene.tsv)
- Cell barcode file (barcode.tsv)
- Surface protein feature file (protein.tsv)
- Pretrained PBMC scMM mixture-of-experts multimodal VAE model

## Outputs

- Predicted abundance values for 24 shared surface proteins per cell (~30,000 BMNC cells)
- Per-protein Jaccard index scores for scMM and Seurat baseline written to crossmodal_prediction_ji.csv

## How to apply

Load the target batch's CITE-seq matrices (Protein_count.mtx, gene.tsv, barcode.tsv, protein.tsv) from the scMM package data directory, following the README folder layout with RNA-seq and CITE-seq subfolders. Apply the pretrained PBMC scMM mixture-of-experts multimodal VAE to generate crossmodal predictions of the 24 shared surface proteins for ~30,000 BMNC cells from transcriptome-only input. Run the Seurat anchor-based transfer baseline on the same BMNC data to predict the same 24 proteins. For each method and protein, compute the Jaccard index between the predicted and measured top-expressing cell sets, aggregate per-protein JI scores for scMM and Seurat, and write them to crossmodal_prediction_ji.csv. scMM is expected to outperform the Seurat state-of-the-art baseline, reproducing the paper's reported superiority on crossmodal prediction.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE used to generate crossmodal predictions of surface proteins from transcriptome-only input) — https://github.com/kodaim1115/scMM
- **Seurat** (Anchor-based transfer baseline for cross-modal surface protein prediction, used for Jaccard index comparison against scMM)
- **R** (Environment for downstream analysis of scMM outputs (R/tutorial.R) and for the Seurat baseline and Jaccard index computation)

## Examples

```
source('R/scMM.R'); pred <- crossmodal_prediction(model = pretrained_pbmc_scmm, rna = readMM('data/BMNC/RNA-seq/RNA_count.mtx'), target = 'protein')
```

## Evaluation signals

- crossmodal_prediction_ji.csv is produced with per-protein Jaccard index scores for both scMM and the Seurat baseline across all 24 shared surface proteins
- Aggregate Jaccard index for scMM exceeds that of the Seurat anchor-based transfer baseline, reproducing the reported superiority
- Predictions are generated for all ~30,000 BMNC cells from transcriptome-only input, confirming the model ran in inference mode without protein input leakage
- The same 24 surface proteins are evaluated for both methods, ensuring a like-for-like comparison

## Limitations

- The pretrained model was trained on PBMC data; cross-batch transfer to BMNC assumes shared cell types and 24 shared surface proteins between batches
- The scMM tutorial/toy BMNC data are randomly subsampled to 15,000 cells with the 5000 most variable genes selected for transcriptome data
- Jaccard index only evaluates agreement on the top-expressing cell sets per protein, not full abundance prediction accuracy

## Evidence
