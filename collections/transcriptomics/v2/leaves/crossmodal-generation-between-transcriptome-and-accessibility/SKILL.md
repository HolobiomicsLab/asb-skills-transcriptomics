---
name: crossmodal-generation-between-transcriptome-and-accessibility
description: Use when paired single-cell multiome data (e.g. SHARE-seq RNA + ATAC
  count matrices) are available and you need to predict gene expression from chromatin
  accessibility or vice versa, with a quantitative benchmark against GAM-based integration
  baselines.
when_to_use_negative:
- Data are unpaired single-cell transcriptome and accessibility measurements, so no
  cell-wise conditioning target exists for crossmodal generation
- The goal is only clustering or joint latent representation of multimodal data without
  generating one modality from the other
- Input is a precomputed gene activity matrix rather than raw per-peak accessibility
  counts, since the GAM is the baseline being compared against, not the input
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE trained on raw RNA and ATAC counts and used
    to perform the crossmodal generation between transcriptome and accessibility
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Source of the mixture-of-experts multimodal VAE training utilities adopted
    by scMM
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Downstream analysis of scMM outputs via R/tutorial.R for evaluating generation
    results
  repo: https://github.com/kodaim1115/scMM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:905c45159ff1d1ff23a7b81697830b11026552e8cca7b7a6bd1aaaeb61b22b26
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# crossmodal-generation-between-transcriptome-and-accessibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train scMM, a mixture-of-experts multimodal variational autoencoder, on paired single-cell RNA-seq and ATAC-seq raw counts (e.g. mouse skin SHARE-seq) and use it to bidirectionally generate one modality from the other, benchmarking against the conventional gene activity matrix (+2 kb upstream of TSS) baseline.

## When to use

Use when paired single-cell multiome data (e.g. SHARE-seq RNA + ATAC count matrices) are available and you need to predict gene expression from chromatin accessibility or vice versa, with a quantitative benchmark against GAM-based integration baselines.

## When NOT to use

- Data are unpaired single-cell transcriptome and accessibility measurements, so no cell-wise conditioning target exists for crossmodal generation
- The goal is only clustering or joint latent representation of multimodal data without generating one modality from the other
- Input is a precomputed gene activity matrix rather than raw per-peak accessibility counts, since the GAM is the baseline being compared against, not the input

## Inputs

- Paired single-cell RNA-seq raw count matrix (features x barcodes, .mtx) with gene.tsv and barcode.tsv
- Paired single-cell ATAC-seq raw count matrix (peaks x barcodes, .mtx) with peak.tsv and barcode.tsv
- Trained scMM model checkpoint (mixture-of-experts multimodal VAE)
- Gene annotation with transcription start sites (for constructing the +2 kb upstream GAM baseline)

## Outputs

- Per-cell predicted gene expression profiles generated from chromatin accessibility
- Per-cell predicted accessibility profiles generated from gene expression
- crossmodal_generation_metrics.csv comparing scMM vs GAM on Pearson correlation and Jaccard index

## How to apply

Load the paired count matrices in the scMM input layout: features x barcodes mtx files with gene.tsv (transcriptome), peak.tsv (accessibility), and barcode.tsv annotations, stored in RNA-seq/ and ATAC-seq/ folders. Train the scMM model, a mixture-of-experts multimodal VAE that models raw counts of each modality with modality-appropriate probability distributions, adopting training utilities from the MMVAE repository to enable end-to-end learning. Condition the trained model on accessibility inputs to decode predicted expression profiles and on expression inputs to decode predicted accessibility profiles (crossmodal generation in both directions). Evaluate the accessibility-to-transcriptome direction with Pearson correlation between predicted and observed expression and with the Jaccard index of top-ranked gene sets. Compute the conventional baseline as a gene activity matrix aggregating accessibility over peaks within +2 kb upstream of the TSS (the GAM used by LIGER/Seurat integration), apply the same metrics, and compare scMM against it, writing per-metric results to crossmodal_generation_metrics.csv.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE trained on raw RNA and ATAC counts and used to perform the crossmodal generation between transcriptome and accessibility) — https://github.com/kodaim1115/scMM
- **MMVAE** (Source of the mixture-of-experts multimodal VAE training utilities adopted by scMM) — https://github.com/iffsid/mmvae
- **R** (Downstream analysis of scMM outputs via R/tutorial.R for evaluating generation results) — https://github.com/kodaim1115/scMM

## Examples

```
python colab_tutorial.ipynb  # or: python main.py with RNA-seq/ and ATAC-seq/ count.mtx + gene.tsv/peak.tsv + barcode.tsv under data/, then evaluate with Rscript R/tutorial.R
```

## Evaluation signals

- Pearson correlation between predicted and observed expression in the accessibility-to-transcriptome direction should be high, matching the paper's reported high reconstruction correlations on SHARE-seq data
- Jaccard index of top-ranked predicted gene sets against observed top-ranked gene sets should exceed the GAM (+2 kb upstream of TSS) baseline
- crossmodal_generation_metrics.csv contains paired scMM vs GAM values for both correlation and Jaccard metrics
- Input count matrices parse correctly in the scMM layout (mtx with matching gene.tsv/peak.tsv and barcode.tsv dimensions)

## Limitations

- Crossmodal generation by scMM was validated on paired datasets; performance on unpaired or surface-protein modalities may differ from the SHARE-seq accessibility-to-transcriptome benchmark
- The GAM baseline aggregates only peaks within +2 kb upstream of the TSS, so it captures distal enhancer effects poorly — the comparison is specifically against this conventional baseline used by LIGER and Seurat
- Like other deep generative models, scMM has limited interpretability, which the authors address with a pseudocell generation strategy rather than removing the limitation

## Evidence
