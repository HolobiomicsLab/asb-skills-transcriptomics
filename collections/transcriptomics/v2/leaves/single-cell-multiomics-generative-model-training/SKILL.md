---
name: single-cell-multiomics-generative-model-training
description: Use when you have paired single-cell multiomics raw count matrices (RNA
  and ATAC or CITE-seq) in mtx format with gene/peak/barcode annotation files and
  need joint representations or crossmodal prediction between modalities.
when_to_use_negative:
- You have only a single modality (unpaired RNA-seq or ATAC-seq) — scMM's joint latent
  inference requires multimodal paired data.
- Your goal is clustering/interpretability of a single dataset rather than crossmodal
  generation or benchmarking against a GAM baseline.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scMM
  role: Mixture-of-experts multimodal deep generative model trained on paired count
    data for joint representation learning and crossmodal generation
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Source of training utilities/code adopted for the mixture-of-experts VAE training
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Downstream analysis of scMM outputs via R/tutorial.R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:3dbb0efff4d40b1a1e52c0c6f3a7b799a85a507ba83c350ebb4ecfb39e49c1bf
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-multiomics-generative-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train scMM, a mixture-of-experts multimodal variational autoencoder that learns joint latent representations from paired single-cell multiomics count data (e.g. RNA-seq and ATAC-seq), enabling crossmodal generation between transcriptome and chromatin accessibility.

## When to use

Use when you have paired single-cell multiomics raw count matrices (RNA and ATAC or CITE-seq) in mtx format with gene/peak/barcode annotation files and need joint representations or crossmodal prediction between modalities.

## When NOT to use

- You have only a single modality (unpaired RNA-seq or ATAC-seq) — scMM's joint latent inference requires multimodal paired data.
- Your goal is clustering/interpretability of a single dataset rather than crossmodal generation or benchmarking against a GAM baseline.

## Inputs

- Paired single-cell RNA-seq raw count matrix (features x barcodes, mtx format) with gene.tsv and barcode.tsv
- Paired single-cell ATAC-seq raw count matrix (features x barcodes, mtx format) with peak.tsv and barcode.tsv
- Optional CITE-seq protein count matrix with protein.tsv and barcode.tsv (e.g. BMNC toy data of 15,000 subsampled cells, top 5000 variable genes)

## Outputs

- Trained scMM mixture-of-experts multimodal VAE model
- Crossmodal-generated profiles (predicted expression from accessibility, predicted accessibility from expression) per cell
- Per-metric comparison of scMM vs GAM baseline (Pearson correlation, Jaccard index) e.g. written to crossmodal_generation_metrics.csv

## How to apply

Organize the count matrices per modality in folders named `RNA-seq` and `ATAC-seq` (or `CITE-seq`), each containing the count mtx plus `gene.tsv` (or `peak.tsv`) and `barcode.tsv` annotations, as required by scMM's data loading layout. Train scMM, a mixture-of-experts multimodal variational autoencoder that achieves end-to-end learning by modeling raw count data in each modality with modality-appropriate probability distributions, adopting training utilities from the MMVAE repository. For ATAC-seq input, peaks within +2 kb upstream of the TSS also define the conventional gene activity matrix (GAM) baseline used by LIGER/Seurat integration, against which scMM crossmodal generation should be benchmarked. After training, condition the model on chromatin accessibility inputs to decode predicted gene expression (accessibility-to-transcriptome generation) and on expression inputs to decode accessibility profiles. Assess reconstruction with Pearson correlation between predicted and observed expression and Jaccard index of top-ranked gene sets, comparing scMM to the GAM baseline.

## Related tools

- **scMM** (Mixture-of-experts multimodal deep generative model trained on paired count data for joint representation learning and crossmodal generation) — https://github.com/kodaim1115/scMM
- **MMVAE** (Source of training utilities/code adopted for the mixture-of-experts VAE training) — https://github.com/iffsid/mmvae
- **R** (Downstream analysis of scMM outputs via R/tutorial.R)

## Examples

```
python train.py --modality1 RNA-seq --modality2 ATAC-seq; then run Rscript R/tutorial.R for downstream analysis of scMM outputs
```

## Evaluation signals

- Accessibility-to-transcriptome reconstruction achieves high Pearson correlation between predicted and observed gene expression
- Jaccard index of top-ranked gene sets between predicted and observed expression exceeds the GAM (+2 kb upstream of TSS) baseline used by LIGER/Seurat
- Input matrices load correctly in scMM's expected folder layout (RNA_count.mtx with gene.tsv and barcode.tsv; ATAC peaks with peak.tsv)
- Transcriptome-to-transcriptome reconstruction remains high, confirming the model did not degrade within-modality performance while learning crossmodal mapping
- Trained model reproduces crossmodal generation results consistent with the paper's finding that scMM outperforms conventional integration baselines

## Limitations

- Training GPU-accelerated scMM is demonstrated via colab_tutorial.ipynb; local setups must supply their own GPU environment.
- The repository tutorial uses toy data (CITE-seq BMNC, 15,000 subsampled cells, top 5000 variable genes); reproducing SHARE-seq ATAC+RNA results requires obtaining the paired mouse skin dataset separately.
- Only high-level training and downstream-analysis entry points are documented; modality-specific likelihood scaling or latent-dimension hyperparameters must be adapted from MMVAE conventions.

## Evidence

- [abstract] Crossmodal generation by scMM leads to more precise prediction and data integration compared with state-of-the-art and conventional approaches: [evidence span withheld — anchor status: not_found]
