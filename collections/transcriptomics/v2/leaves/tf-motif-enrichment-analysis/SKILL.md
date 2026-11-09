---
name: tf-motif-enrichment-analysis
description: Use when scMM latent dimensions from single-cell multiomics (e.g. CITE-seq
  transcriptome + surface protein, or ATAC-seq accessibility) lack biological interpretation
  and you need to link each latent dimension to enriched TF motifs via decoded pseudocells.
when_to_use_negative:
- The trained model is not scMM and you have no access to its decoder, since pseudocell
  decoding requires the mixture-of-experts decoder
- The analysis goal is crossmodal prediction or data integration between modalities
  rather than interpreting what individual latent dimensions encode
- Input is raw count data that has not yet been trained through scMM — motif enrichment
  here operates on decoded pseudocell accessibility, not primary counts
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scMM
  role: Provides the trained mixture-of-experts multimodal VAE whose latent dimensions
    are traversed and decoded into pseudocells for motif enrichment testing
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Language of the downstream analysis tutorial (R/tutorial.R) that demonstrates
    loading scMM outputs and running the pseudocell/motif workflow
- name: MMVAE
  role: Original mixture-of-experts VAE repository from which the pseudocell generation
    / latent traversal code approach was adopted
  repo: https://github.com/iffsid/mmvae
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:99c8e33cd677b6c7b245026c183a5a0cedd22d5bdf8084c8a04f6e47c61b2d57
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# tf-motif-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Associates individual latent dimensions of the trained scMM mixture-of-experts model with transcription factor regulatory programs by aggregating chromatin accessibility peaks from pseudocells generated along each latent dimension and testing those peaks for transcription factor motif enrichment.

## When to use

Use when scMM latent dimensions from single-cell multiomics (e.g. CITE-seq transcriptome + surface protein, or ATAC-seq accessibility) lack biological interpretation and you need to link each latent dimension to enriched TF motifs via decoded pseudocells.

## When NOT to use

- The trained model is not scMM and you have no access to its decoder, since pseudocell decoding requires the mixture-of-experts decoder
- The analysis goal is crossmodal prediction or data integration between modalities rather than interpreting what individual latent dimensions encode
- Input is raw count data that has not yet been trained through scMM — motif enrichment here operates on decoded pseudocell accessibility, not primary counts

## Inputs

- Trained scMM model latent embeddings and mixture-of-experts decoder
- RNA count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv
- CITE-seq protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv
- Chromatin accessibility peak features (peak.tsv) for ATAC-seq modalities

## Outputs

- Per-dimension pseudocell multimodal feature tables (gene expression, surface protein counts, accessibility peaks) as TSV files
- latent_dimension_enriched_motifs.tsv with latent dimension, motif, and enrichment significance fields
- Recovered dimension-to-regulatory-program associations

## How to apply

Load the trained scMM model outputs (latent embeddings and mixture-of-experts decoder) together with the multimodal inputs — RNA_count.mtx with gene.tsv and barcode.tsv, Protein_count.mtx with protein.tsv, or peak.tsv for chromatin accessibility — as arranged in the data/BMNC folder structure demonstrated in R/tutorial.R. For each latent dimension in turn, sweep its value across evenly spaced levels while holding the other latent dimensions fixed, decoding each point into a series of pseudocells with the scMM decoder (an approach adopted from the MMVAE repository). Extract the generated multimodal features per pseudocell series — gene expression, surface protein counts, and accessibility peaks — and aggregate the accessibility peaks along each dimension's traversal. Run transcription factor motif enrichment testing on the aggregated peaks per dimension to obtain enriched motifs, then write per-dimension pseudocell feature tables and motif results as TSV files, with the combined motif table saved as latent_dimension_enriched_motifs.tsv. This compensates for the limited interpretability of the deep generative model by experimentally discovering multimodal regulatory programs associated with latent dimensions.

## Related tools

- **scMM** (Provides the trained mixture-of-experts multimodal VAE whose latent dimensions are traversed and decoded into pseudocells for motif enrichment testing) — https://github.com/kodaim1115/scMM
- **R** (Language of the downstream analysis tutorial (R/tutorial.R) that demonstrates loading scMM outputs and running the pseudocell/motif workflow)
- **MMVAE** (Original mixture-of-experts VAE repository from which the pseudocell generation / latent traversal code approach was adopted) — https://github.com/iffsid/mmvae

## Examples

```
Rscript R/tutorial.R  # downstream analysis: traverse each latent dimension, decode pseudocells, aggregate peaks, and test TF motif enrichment -> latent_dimension_enriched_motifs.tsv
```

## Evaluation signals

- latent_dimension_enriched_motifs.tsv exists and parses as valid TSV with fields for latent dimension, motif, and enrichment significance
- All enrichment scores in the motif table are finite (no NaN/Inf values)
- Each latent dimension yields a pseudocell series decoded with the other latent dimensions held fixed, confirming independent per-dimension traversal
- Recovered dimension-to-regulatory-program associations are consistent with the multimodal regulatory programs the paper reports as discovered by the pseudocell generation strategy
- Per-dimension feature tables contain generated gene expression, CITE-seq protein counts, and accessibility peaks matching the input modalities

## Limitations

- Deep generative model latent dimensions have limited inherent interpretability — motif enrichment is a post-hoc compensation strategy, not a built-in guarantee
- The approach relies on the trained model; toy tutorial data are randomly subsampled 15,000 CITE-seq BMNC cells with the 5000 most variable genes, so programs may differ on other datasets
- The pseudocell traversal approach was adopted from the MMVAE repository, so it inherits that framework's assumptions about latent traversal

## Evidence

- [abstract] pseudocell generation workflow step: [evidence span withheld — anchor status: not_found]
