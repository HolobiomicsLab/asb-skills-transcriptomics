---
name: single-cell-multiomics-data-integration
description: Use when you have paired single-cell multiomics count matrices (e.g.
  RNA + surface protein CITE-seq, or RNA + ATAC peaks) and need joint dimensionality
  reduction, cross-modal prediction, or interpretable latent dimensions.
when_to_use_negative:
- Data modalities are unpaired (not measured in the same cells) — the mixture-of-experts
  joint representation requires aligned paired counts
- Input is already a reduced feature/embedding table rather than raw integer counts
  — scMM models raw count data per modality with modality-specific distributions
- Only a single modality is available — cross-modal generation and multimodal joint
  representation require at least two modalities
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE trained on paired raw count data to extract
    joint representations and perform cross-modal generation
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Repository from which scMM's variational mixture-of-experts code was adopted
  repo: https://github.com/iffsid/mmvae
- name: totalVI
  role: State-of-the-art comparison method (scvi-tools) trained on the identical dataset
    for ELBO-based benchmarking
- name: R
  role: Downstream analysis of scMM outputs via R/tutorial.R and vignette
  repo: http://htmlpreview.github.io/?https://github.com/kodaim1115/test/blob/master/tutorial.html
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:39066283d8aef3510739cc2dfc81248a3df5a59265eeef706c801091aaba6c87
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
  - totalVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-multiomics-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrate paired single-cell multiomics data (transcriptome & surface proteins via CITE-seq, or transcriptome & chromatin accessibility) with scMM, a mixture-of-experts multimodal variational autoencoder that models raw count data per modality with modality-specific probability distributions to extract interpretable joint representations and enable cross-modal generation.

## When to use

Use when you have paired single-cell multiomics count matrices (e.g. RNA + surface protein CITE-seq, or RNA + ATAC peaks) and need joint dimensionality reduction, cross-modal prediction, or interpretable latent dimensions.

## When NOT to use

- Data modalities are unpaired (not measured in the same cells) — the mixture-of-experts joint representation requires aligned paired counts
- Input is already a reduced feature/embedding table rather than raw integer counts — scMM models raw count data per modality with modality-specific distributions
- Only a single modality is available — cross-modal generation and multimodal joint representation require at least two modalities

## Inputs

- RNA count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv
- CITE-seq surface protein count matrix (Protein_count.mtx, 224 surface proteins) with protein.tsv and barcode.tsv
- ATAC-seq peak count matrix with peak.tsv (optional third modality)
- Trained scMM model outputs (latent embeddings and decoder) for downstream analysis

## Outputs

- Joint latent representation (embeddings) of paired single-cell multiomics data
- Cross-modal generated data (e.g. protein counts predicted from transcriptome)
- Per-dimension pseudocell feature tables and transcription factor motif enrichment results (latent_dimension_enriched_motifs.tsv)
- ELBO benchmark table (elbo_values.csv) comparing scMM against totalVI

## How to apply

Organize count matrices into modality folders: RNA-seq (RNA_count.mtx, gene.tsv, barcode.tsv) and CITE-seq (Protein_count.mtx, protein.tsv, barcode.tsv), or ATAC-seq with peak.tsv, keeping raw integer counts so each modality is modeled with its own probability distribution for end-to-end learning. Train scMM — whose code is adopted from the MMVAE repository (iffsid/mmvae) and leverages a mixture-of-experts multimodal VAE — on the paired data to obtain joint latent embeddings. For benchmarking, train totalVI (scvi-tools) on the identical dataset and compare evidence lower bound (ELBO) values as the dimensionality-reduction quality metric. For interpretability, generate pseudocells by independently traversing each latent dimension while holding the rest fixed, decoding each point with the mixture-of-experts decoder, and perform transcription factor motif enrichment on aggregated accessibility peaks per dimension. Follow R/tutorial.R for downstream analysis of scMM outputs, or colab_tutorial.ipynb for GPU execution on Google Colab.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE trained on paired raw count data to extract joint representations and perform cross-modal generation) — https://github.com/kodaim1115/scMM
- **MMVAE** (Repository from which scMM's variational mixture-of-experts code was adopted) — https://github.com/iffsid/mmvae
- **totalVI** (State-of-the-art comparison method (scvi-tools) trained on the identical dataset for ELBO-based benchmarking)
- **R** (Downstream analysis of scMM outputs via R/tutorial.R and vignette) — http://htmlpreview.github.io/?https://github.com/kodaim1115/test/blob/master/tutorial.html

## Examples

```
python main.py --model mnist_svhn  # MMVAE-style training; for scMM follow colab_tutorial.ipynb, then Rscript R/tutorial.R for downstream analysis
```

## Evaluation signals

- scMM ELBO is similar to totalVI's ELBO when both are trained and evaluated on the same PBMC CITE-seq dataset (the reported finding)
- Downstream outputs (latent_dimension_enriched_motifs.tsv) exist, are valid TSV with fields for latent dimension, motif, and enrichment significance, and all enrichment scores are finite
- Recovered dimension-to-regulatory-program associations from pseudocell traversal are consistent with the multimodal regulatory programs reported in the paper
- Input folder structure matches the required layout: RNA-seq/ (RNA_count.mtx, gene.tsv, barcode.tsv) and CITE-seq/ (Protein_count.mtx, protein.tsv, barcode.tsv), with barcodes aligned across modalities
- Raw integer counts are preserved in inputs (no normalization) since each modality is modeled with its own count likelihood

## Limitations

- Deep generative latent dimensions have limited interpretability, which must be compensated by the pseudocell generation strategy
- The tutorial uses toy data generated from CITE-seq bone marrow mononuclear cell (BMNC) data with randomly subsampled 15,000 cells and the most variable 5000 genes — expect to adapt inputs for other datasets
- Numeric values in benchmark-style analyses can vary across recomputation; relative performance against baselines is the reliable signal (noted in the adopted MMVAE repository)

## Evidence
