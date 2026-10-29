---
name: cross-modal-decoder-generation
description: 'Use when a trained scMM model''s latent dimensions need interpretation:
  sweep each latent dimension independently through the mixture-of-experts decoder
  to link genes, surface proteins, and chromatin accessibility peaks to individual
  dimensions.'
when_to_use_negative:
- The goal is cross-modal prediction between modalities from real cells (e.g. imputing
  protein from RNA) rather than interpreting what each latent dimension encodes —
  use scMM's crossmodal generation mode instead
- No trained scMM decoder is available (e.g. only raw count matrices without a fitted
  model) — the traversal requires the trained generative model
- The model is the generic MMVAE on non-single-cell modalities (e.g. MNIST-SVHN, CUB
  image-caption); this skill's feature extraction and motif enrichment steps are specific
  to single-cell multiomics data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMM
  role: Provides the trained mixture-of-experts multimodal VAE whose latent dimensions
    are independently traversed and decoded into pseudocells
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Used for the downstream analysis of scMM outputs demonstrated in R/tutorial.R,
    including pseudocell feature extraction and motif enrichment assembly
  repo: https://github.com/kodaim1115/scMM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:6bcd2d08c153756cc5a6e794c159ebdb55877840ab11a314030e11f15b404369
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
  - scMM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-modal-decoder-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate pseudocells by independently traversing each latent dimension of a trained scMM mixture-of-experts model and decoding each swept value with the multimodal decoder, compensating for the limited interpretability of deep generative latent spaces by associating multimodal features and regulatory programs with individual dimensions.

## When to use

Use when a trained scMM model's latent dimensions need interpretation: sweep each latent dimension independently through the mixture-of-experts decoder to link genes, surface proteins, and chromatin accessibility peaks to individual dimensions.

## When NOT to use

- The goal is cross-modal prediction between modalities from real cells (e.g. imputing protein from RNA) rather than interpreting what each latent dimension encodes — use scMM's crossmodal generation mode instead
- No trained scMM decoder is available (e.g. only raw count matrices without a fitted model) — the traversal requires the trained generative model
- The model is the generic MMVAE on non-single-cell modalities (e.g. MNIST-SVHN, CUB image-caption); this skill's feature extraction and motif enrichment steps are specific to single-cell multiomics data

## Inputs

- Trained scMM model outputs: latent embeddings and mixture-of-experts decoder
- Multimodal count matrices and feature files: RNA_count.mtx with gene.tsv and barcode.tsv; Protein_count.mtx with protein.tsv and barcode.tsv (ATAC-seq peaks as peak.tsv when applicable)
- Latent dimension list from the trained scMM model

## Outputs

- Per-dimension pseudocell series of decoded multimodal features (gene expression, surface protein counts, accessibility peaks)
- Per-dimension transcription factor motif enrichment table saved as latent_dimension_enriched_motifs.tsv (TSV with latent dimension, motif, and enrichment significance fields)
- Interpretation mapping of individual latent dimensions to multimodal regulatory programs

## How to apply

Load the trained scMM latent embeddings and mixture-of-experts decoder together with the multimodal inputs (gene.tsv, barcode.tsv, Protein_count.mtx, protein.tsv) as laid out in R/tutorial.R. For each latent dimension in turn, sweep its value across evenly spaced levels while holding all other dimensions fixed, and decode each point into a series of pseudocells; this strategy is adopted from the original MMVAE paper. Extract per-pseudocell gene expression, CITE-seq surface protein counts, and, where present, chromatin accessibility peaks from the decoded series. Aggregate accessibility peaks along each dimension's traversal and run transcription factor motif enrichment testing to obtain motifs associated with that dimension. Assemble per-dimension feature tables and write motif results to latent_dimension_enriched_motifs.tsv, then check the recovered dimension-to-regulatory-program associations against the multimodal regulatory programs the paper reports.

## Related tools

- **scMM** (Provides the trained mixture-of-experts multimodal VAE whose latent dimensions are independently traversed and decoded into pseudocells) — https://github.com/kodaim1115/scMM
- **R** (Used for the downstream analysis of scMM outputs demonstrated in R/tutorial.R, including pseudocell feature extraction and motif enrichment assembly) — https://github.com/kodaim1115/scMM

## Examples

```
Rscript R/tutorial.R  # downstream analysis of trained scMM outputs: latent dimension traversal, pseudocell decoding, and motif enrichment to latent_dimension_enriched_motifs.tsv
```

## Evaluation signals

- latent_dimension_enriched_motifs.tsv exists and is valid TSV with fields for latent dimension, motif, and enrichment significance
- All enrichment scores in the motif table are finite (no NaN/Inf values)
- Every latent dimension of the trained model has a corresponding decoded pseudocell series covering the full sweep range
- Decoded pseudocells contain values for all input modalities (gene expression, CITE-seq protein counts, accessibility peaks where applicable)
- Recovered dimension-to-regulatory-program associations are consistent with the multimodal regulatory programs reported in the paper

## Limitations

- Deep generative model latent dimensions have limited intrinsic interpretability; the pseudocell traversal is a post hoc strategy that compensates for, but does not remove, this limitation
- The approach is adopted from the original MMVAE paper and assumes latent dimensions are sufficiently disentangled for independent traversal to produce interpretable feature changes
- Motif enrichment from aggregated accessibility peaks depends on having an ATAC-seq modality; CITE-seq-only setups (transcriptome & surface proteins) yield no chromatin-based regulatory interpretation

## Evidence

- [abstract] Pseudocell generation compensates for limited interpretability: "The pseudocell generation strategy of scMM compensates for the limited interpretability of deep learning models, and the proposed approach experimentally discovered multimodal regulatory programs"
- [methods] Codes adopted from MMVAE: "Codes were adopted from the [MMVAE repository](https://github.com/iffsid/mmvae)"
- [methods] Downstream analysis tutorial in R: "Tutorial on downstream analysisfor scMM outputs can be found at `R/tutorial.R`"
- [readme] Input file layout for multimodal data: "RNA and protein count matrix should be stored in folder named `RNA-seq` and `CITE-seq` accomapnied with feature information stored in `gene.tsv` and `protein.tsv`"
- [readme] Mixture-of-experts basis of the model: "It is based on a mixture-of-experts multimodal deep generative model and achieves end-to-end learning by modeling raw count data in each modality based on different probability distributions."
