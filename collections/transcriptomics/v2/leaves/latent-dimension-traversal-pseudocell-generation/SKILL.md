---
name: latent-dimension-traversal-pseudocell-generation
description: 'Use when a trained scMM model''s latent space needs biological interpretation:
  to link individual latent dimensions to gene expression programs, surface protein
  levels, or chromatin accessibility motifs from CITE-seq/Atlas-seq multimodal single-cell
  data.'
when_to_use_negative:
- The model has not yet been trained — pseudocell generation requires fitted latent
  embeddings and a trained decoder, not raw count data
- The goal is crossmodal prediction or data integration between modalities rather
  than interpreting what individual latent dimensions encode
- Input data lacks multiple modalities and only single-modality interpretation is
  needed without regulatory program discovery
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3673
tools:
- name: scMM
  role: Provides the trained mixture-of-experts multimodal VAE whose latent dimensions
    are traversed and whose decoder generates the pseudocells
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Used for the downstream analysis of scMM outputs (R/tutorial.R) that implements
    the pseudocell generation workflow
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Original mixture-of-experts VAE repository from which the latent traversal
    code and approach were adopted
  repo: https://github.com/iffsid/mmvae
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:f0f7f11d63b766e610bafe91c1a80b352255e51d7be88d1c323f288e23fedc09
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# latent-dimension-traversal-pseudocell-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates scMM pseudocells by independently sweeping each latent dimension of the trained mixture-of-experts multimodal VAE while holding other dimensions fixed, then decodes each point into multimodal features to associate regulatory programs with individual latent dimensions. This compensates for the limited interpretability of deep generative models.

## When to use

Use when a trained scMM model's latent space needs biological interpretation: to link individual latent dimensions to gene expression programs, surface protein levels, or chromatin accessibility motifs from CITE-seq/Atlas-seq multimodal single-cell data.

## When NOT to use

- The model has not yet been trained — pseudocell generation requires fitted latent embeddings and a trained decoder, not raw count data
- The goal is crossmodal prediction or data integration between modalities rather than interpreting what individual latent dimensions encode
- Input data lacks multiple modalities and only single-modality interpretation is needed without regulatory program discovery

## Inputs

- Trained scMM model outputs (latent embeddings and mixture-of-experts decoder)
- Multimodal count matrices: RNA_count.mtx, Protein_count.mtx (Matrix Market format)
- Feature files: gene.tsv, protein.tsv (and peak.tsv for ATAC-seq modality)
- Single-cell barcodes: barcode.tsv

## Outputs

- Per-dimension pseudocell series of decoded multimodal features (gene expression, surface protein counts, accessibility peaks)
- Per-dimension transcription factor motif enrichment table, e.g. latent_dimension_enriched_motifs.tsv
- Per-dimension pseudocell feature tables written as TSV files

## How to apply

Load the trained scMM model outputs (latent embeddings and mixture-of-experts decoder) together with the input modalities (gene.tsv, barcode.tsv, Protein_count.mtx, protein.tsv) as demonstrated in R/tutorial.R. For each latent dimension in turn, sweep its value across evenly spaced levels while keeping all remaining dimensions fixed, decoding each point with the scMM decoder into a series of pseudocells. Extract gene expression, surface protein counts, and chromatin accessibility peaks per pseudocell series; aggregate peaks along each traversal and run transcription factor motif enrichment to identify motifs associated with that dimension. The traversal approach is adopted from the original MMVAE paper (iffsid/mmvae), from which the scMM code was adapted. Write per-dimension pseudocell feature tables and motif results as TSV, e.g. latent_dimension_enriched_motifs.tsv.

## Related tools

- **scMM** (Provides the trained mixture-of-experts multimodal VAE whose latent dimensions are traversed and whose decoder generates the pseudocells) — https://github.com/kodaim1115/scMM
- **R** (Used for the downstream analysis of scMM outputs (R/tutorial.R) that implements the pseudocell generation workflow) — https://github.com/kodaim1115/scMM
- **MMVAE** (Original mixture-of-experts VAE repository from which the latent traversal code and approach were adopted) — https://github.com/iffsid/mmvae

## Examples

```
Rscript R/tutorial.R  # downstream analysis of trained scMM outputs, including latent dimension traversal for pseudocell generation
```

## Evaluation signals

- latent_dimension_enriched_motifs.tsv exists and parses as valid TSV with fields for latent dimension, motif, and enrichment significance
- All enrichment scores in the motif table are finite (no NA/Inf values)
- Each latent dimension yields a pseudocell series decoded into all input modalities (gene expression, protein counts, accessibility peaks)
- Recovered dimension-to-regulatory-program associations are consistent with the multimodal regulatory programs reported in the paper

## Limitations

- Deep generative model latent dimensions lack intrinsic interpretability — the traversal is a post-hoc strategy that compensates for this, not a guarantee of disentangled dimensions
- Pseudocells are model-generated rather than observed cells, so decoded features depend on decoder calibration and mixture-of-experts likelihood assumptions per modality
- Motif enrichment on aggregated peaks is indirect: dimension-to-motif associations can vary if the model is retrained or the latent sweep levels change

## Evidence
