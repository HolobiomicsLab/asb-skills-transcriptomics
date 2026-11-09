---
name: seurat-anchor-based-transfer-benchmarking
description: Use when evaluating crossmodal prediction of surface proteins from transcriptome-only
  input on a CITE-seq batch, and you need the Seurat anchor-based baseline (Jaccard
  index per protein) to compare against a deep generative model such as scMM.
when_to_use_negative:
- The goal is joint clustering or interpretability of paired multiomics data rather
  than benchmarking crossmodal protein prediction from transcriptome-only input
- Only one modality is available and there is no measured protein count matrix to
  compute measured-vs-predicted Jaccard index
- The target batch has few or none of the 24 surface proteins shared with the pretrained
  training data
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Anchor-based transfer baseline used to predict BMNC surface proteins from
    transcriptome data, benchmarked against scMM crossmodal prediction via Jaccard
    index
- name: scMM
  role: Mixture-of-experts multimodal VAE whose pretrained PBMC model provides the
    comparator crossmodal predictions on BMNC data
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Environment for the Seurat baseline and downstream analysis of scMM outputs
    (R/tutorial.R)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:d04f2e22e011538093a0ea059e5125a69796947e71973359366d65792ae819ad
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
yes_rate: 0.42857142857142855
n_claims: 7
---

# Seurat anchor-based transfer benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Baseline skill for benchmarking crossmodal surface-protein prediction: runs Seurat's anchor-based transfer on CITE-seq data (e.g., the BMNC batch from the scMM study) and scores predicted vs. measured protein expression with per-protein Jaccard index against the scMM mixture-of-experts multimodal VAE predictions.

## When to use

Use when evaluating crossmodal prediction of surface proteins from transcriptome-only input on a CITE-seq batch, and you need the state-of-the-art Seurat anchor-based baseline (Jaccard index per protein) to compare against a deep generative model such as scMM.

## When NOT to use

- The goal is joint clustering or interpretability of paired multiomics data rather than benchmarking crossmodal protein prediction from transcriptome-only input
- Only one modality is available and there is no measured protein count matrix to compute measured-vs-predicted Jaccard index
- The target batch has few or none of the 24 surface proteins shared with the pretrained training data

## Inputs

- BMNC CITE-seq protein count matrix (Protein_count.mtx)
- Feature and barcode files (gene.tsv, protein.tsv, barcode.tsv) in the data/BMNC folder layout
- Pretrained PBMC scMM model predictions for the 24 shared surface proteins (comparator)
- Seurat package in R for anchor-based transfer

## Outputs

- Per-protein Jaccard index scores for the Seurat baseline across the 24 shared surface proteins
- Aggregated comparison table written to crossmodal_prediction_ji.csv (scMM vs. Seurat)

## How to apply

Load the BMNC CITE-seq inputs (Protein_count.mtx, gene.tsv, barcode.tsv, protein.tsv) following the scMM data layout (data/BMNC/RNA-seq and data/BMNC/CITE-seq). Restrict evaluation to the 24 surface proteins shared between the PBMC training data and the BMNC batch, then run the Seurat anchor-based transfer on the same BMNC transcriptome data to predict all ~30,000 cells' protein abundance in parallel with the pretrained PBMC scMM mixture-of-experts multimodal VAE. For each method and each protein, define the set of top-expressing cells among the predicted values and the corresponding set from the measured Protein_count.mtx, and compute the Jaccard index between them. Aggregate per-protein JI scores for Seurat and the comparator (scMM) and write them to crossmodal_prediction_ji.csv so the state-of-the-art baseline and the deep model can be compared directly. The method is judged correct if the per-protein JI table covers all 24 proteins for both methods and reproduces the reported ranking in which scMM's crossmodal generation outperforms the state-of-the-art baseline.

## Related tools

- **Seurat** (Anchor-based transfer baseline used to predict BMNC surface proteins from transcriptome data, benchmarked against scMM crossmodal prediction via Jaccard index)
- **scMM** (Mixture-of-experts multimodal VAE whose pretrained PBMC model provides the comparator crossmodal predictions on BMNC data) — https://github.com/kodaim1115/scMM
- **R** (Environment for the Seurat baseline and downstream analysis of scMM outputs (R/tutorial.R))

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- The JI table in crossmodal_prediction_ji.csv contains exactly one score per method for each of the 24 shared surface proteins
- Jaccard index values fall in [0, 1] for every protein-method pair
- The measured top-expressing cell sets are derived from Protein_count.mtx for the same ~30,000 BMNC barcodes used in prediction
- Reproduces the reported ranking: scMM's per-protein JI exceeds the Seurat anchor-based baseline on the BMNC batch
- Identical protein set and barcode universe are used for both methods so the comparison is paired

## Limitations

- The tutorial dataset is a randomly subsampled 15,000-cell BMNC toy set (top 5000 variable genes), so absolute JI values on the full ~30,000-cell run may differ
- Comparison is limited to the surface proteins shared between the PBMC training data and the BMNC batch (24 proteins); proteins absent from the training set cannot be predicted or scored

## Evidence

- [abstract] Crossmodal generation by scMM leads to more precise prediction and data integration compared with state-of-the-art and conventional approaches: [evidence span withheld — anchor status: not_found]
- [results] Benchmarking showed scMM outperformed the state-of-the-art prediction tool and conventional approaches on crossmodal prediction, demonstrating its superiority for surface protein prediction across batches.: [evidence span withheld — anchor status: not_found]
