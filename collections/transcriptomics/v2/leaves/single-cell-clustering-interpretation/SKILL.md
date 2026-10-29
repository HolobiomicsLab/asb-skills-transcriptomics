---
name: single-cell-clustering-interpretation
description: Use when a trained scMM model's latent dimensions need biological interpretation,
  e.g. associating multimodal regulatory programs with individual latent dimensions
  via pseudocell traversal and TF motif enrichment.
when_to_use_negative:
- The model has not been trained yet — pseudocell traversal requires trained latent
  embeddings and a decoder
- The goal is crossmodal prediction or data integration (e.g. imputing protein from
  RNA) rather than interpreting latent dimensions — scMM's crossmodal generation workflow
  is the appropriate skill
- Input data lacks multiple modalities, so dimension associations cannot be validated
  against multimodal regulatory programs
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3697
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE whose trained latent embeddings and decoder
    are traversed to generate pseudocells
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Downstream analysis of scMM outputs including pseudocell feature extraction
    and motif enrichment, via R/tutorial.R
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Source of the latent-dimension traversal strategy adopted by scMM for pseudocell
    generation
  repo: https://github.com/iffsid/mmvae
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:3fd869574bb18d5ae046bf45ad16b2c1535b561bfe3bc06c247693378a976c49
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-clustering-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret the latent dimensions of scMM's mixture-of-experts multimodal VAE by independently traversing each latent dimension to generate pseudocells, then linking the decoded multimodal features (genes, surface proteins, ATAC-seq peaks) to enriched transcription factor motifs. This compensates for the limited interpretability of deep generative models.

## When to use

Use when a trained scMM model's latent dimensions need biological interpretation, e.g. associating multimodal regulatory programs with individual latent dimensions via pseudocell traversal and TF motif enrichment.

## When NOT to use

- The model has not been trained yet — pseudocell traversal requires trained latent embeddings and a decoder
- The goal is crossmodal prediction or data integration (e.g. imputing protein from RNA) rather than interpreting latent dimensions — scMM's crossmodal generation workflow is the appropriate skill
- Input data lacks multiple modalities, so dimension associations cannot be validated against multimodal regulatory programs

## Inputs

- Trained scMM model outputs (latent embeddings and mixture-of-experts decoder)
- RNA count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv
- CITE-seq protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv
- Optionally ATAC-seq accessibility peaks (peak.tsv) for motif enrichment

## Outputs

- Per-dimension pseudocell multimodal feature tables (gene expression, surface protein counts, accessibility peaks) as TSV
- Per-dimension TF motif enrichment table saved as latent_dimension_enriched_motifs.tsv
- Recovered dimension-to-regulatory-program associations

## How to apply

Load the trained scMM model outputs (latent embeddings and decoder) together with the multimodal inputs (gene.tsv, barcode.tsv, Protein_count.mtx, protein.tsv) following R/tutorial.R. For each latent dimension in turn, sweep its value across evenly spaced levels while holding all other dimensions fixed, and decode each point with the scMM mixture-of-experts decoder into a series of pseudocells — an approach adopted from the original MMVAE paper (iffsid/mmvae). Extract per-pseudocell gene expression, CITE-seq surface protein counts, and chromatin accessibility peaks. Aggregate accessibility peaks along each dimension's traversal and run transcription factor motif enrichment testing to associate motifs with that dimension. Write per-dimension pseudocell feature tables and motif results as TSV files, saving the per-dimension motif table as latent_dimension_enriched_motifs.tsv.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE whose trained latent embeddings and decoder are traversed to generate pseudocells) — https://github.com/kodaim1115/scMM
- **R** (Downstream analysis of scMM outputs including pseudocell feature extraction and motif enrichment, via R/tutorial.R) — https://github.com/kodaim1115/scMM
- **MMVAE** (Source of the latent-dimension traversal strategy adopted by scMM for pseudocell generation) — https://github.com/iffsid/mmvae

## Examples

```
Rscript R/tutorial.R  # downstream analysis of trained scMM outputs: latent-dimension traversal, pseudocell decoding, and motif enrichment
```

## Evaluation signals

- latent_dimension_enriched_motifs.tsv exists and is valid TSV with fields for latent dimension, motif, and enrichment significance
- All enrichment scores in the motif table are finite
- Recovered dimension-to-regulatory-program associations are consistent with the multimodal regulatory programs reported by the paper
- Each pseudocell series contains decoded values for every modality (RNA, protein, and accessibility peaks where present)
- Traversal varies only one latent dimension at a time while all other dimensions remain fixed

## Limitations

- Deep generative models like scMM have limited inherent interpretability of latent dimensions, which is why pseudocell generation is needed at all
- The traversal-based strategy was adopted from the original MMVAE paper and was demonstrated on toy CITE-seq BMNC data (15,000 subsampled cells, 5,000 most variable genes), so associations on new data require their own validation
- Interpretation depends on having chromatin accessibility data to aggregate peaks for motif enrichment; without ATAC-seq, only gene/protein associations per dimension are available

## Evidence
