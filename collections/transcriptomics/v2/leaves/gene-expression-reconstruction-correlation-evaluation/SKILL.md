---
name: gene-expression-reconstruction-correlation-evaluation
description: Use when a trained multimodal generative model (e.g. scMM on paired SHARE-seq
  scRNA-seq/scATAC-seq) has decoded predicted expression profiles that must be quantitatively
  scored against observed transcriptome counts and compared to a GAM baseline.
when_to_use_negative:
- When no paired single-cell multiomics data are available (e.g. only unpaired scATAC-seq),
  since reconstruction accuracy requires observed expression matched by barcode
- When the goal is generic clustering or latent-space interpretation of scMM outputs
  rather than quantifying crossmodal generation quality (use the R/tutorial.R downstream
  analysis instead)
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMM
  role: Mixture-of-experts multimodal VAE used to condition on accessibility inputs
    and decode predicted gene expression profiles for reconstruction scoring
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Source of the mixture-of-experts multimodal VAE training utilities adopted
    by scMM
  repo: https://github.com/iffsid/mmvae
- name: R
  role: Environment for the scMM downstream-analysis tutorial used to evaluate outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:8046efd7cf8c8bd48d121418b95e4e290c4a1c7f041acfc35d3f51eeb82c9a10
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Gene-expression reconstruction correlation evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates accessibility-to-transcriptome crossmodal generation from scMM by computing Pearson correlation between predicted and observed expression plus Jaccard index of top-ranked gene sets, and benchmarks against the gene activity matrix (GAM, +2 kb upstream of TSS) baseline used by LIGER/Seurat-style integration.

## When to use

Use when a trained multimodal generative model (e.g. scMM on paired SHARE-seq scRNA-seq/scATAC-seq) has decoded predicted expression profiles that must be quantitatively scored against observed transcriptome counts and compared to a GAM baseline.

## When NOT to use

- When no paired single-cell multiomics data are available (e.g. only unpaired scATAC-seq), since reconstruction accuracy requires observed expression matched by barcode
- When the goal is generic clustering or latent-space interpretation of scMM outputs rather than quantifying crossmodal generation quality (use the R/tutorial.R downstream analysis instead)

## Inputs

- Paired mouse skin SHARE-seq raw count matrices (features x barcodes in mtx format with gene.tsv and barcode.tsv; peaks in peak.tsv for ATAC-seq)
- scMM model-conditioned decoded expression profiles per cell (accessibility-to-transcriptome generation output)
- Observed scRNA-seq count matrix for the same barcodes
- Gene annotation with TSS coordinates for building the +2 kb upstream GAM baseline

## Outputs

- Per-metric comparison of scMM and GAM (Pearson correlation, Jaccard index of top-ranked gene sets) in crossmodal_generation_metrics.csv
- Reconstruction accuracy summary for the accessibility-to-transcriptome direction

## How to apply

Condition the trained mixture-of-experts MMVAE (scMM) on chromatin accessibility inputs and decode predicted gene expression profiles per cell; scMM achieves end-to-end learning by modeling raw count data in each modality with modality-appropriate probability distributions, so predictions are generated directly from raw counts rather than a smoothed gene activity matrix. Compute Pearson correlation between predicted and observed expression, and a Jaccard index between top-ranked gene sets (predicted vs observed), for the accessibility-to-transcriptome direction. Build the conventional baseline by aggregating ATAC-seq peak accessibility over peaks within +2 kb upstream of each TSS (the GAM used by LIGER and Seurat integration), evaluate the same Pearson/Jaccard metrics on the GAM, and compare. Write the per-metric scMM-vs-GAM comparison to crossmodal_generation_metrics.csv. Rationale: the paper claims crossmodal generation by scMM gives more precise prediction and integration than conventional approaches, which must be verified with these metrics on the same gene sets.

## Related tools

- **scMM** (Mixture-of-experts multimodal VAE used to condition on accessibility inputs and decode predicted gene expression profiles for reconstruction scoring) — https://github.com/kodaim1115/scMM
- **MMVAE** (Source of the mixture-of-experts multimodal VAE training utilities adopted by scMM) — https://github.com/iffsid/mmvae
- **R** (Environment for the scMM downstream-analysis tutorial used to evaluate outputs)

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Pearson correlation between predicted and observed gene expression is computed per cell on the accessibility-to-transcriptome direction and compared to the same metric on the GAM
- Jaccard index of top-ranked gene sets (predicted vs observed) matches or exceeds the GAM baseline value
- crossmodal_generation_metrics.csv contains paired scMM vs GAM entries for each metric with no missing rows
- Input matrices follow the scMM layout (mtx plus gene.tsv/peak.tsv and barcode.tsv in RNA-seq and ATAC-seq folders)
- GAM is restricted to peaks within +2 kb upstream of the TSS, matching the LIGER/Seurat convention

## Limitations

- The conventional GAM baseline aggregates accessibility only within +2 kb upstream of the TSS, so regulatory elements outside that window are ignored and the baseline underestimates distal-enhancer effects
- The MMVAE-inherited analysis scripts note that recomputed projections can vary and numeric values may differ between runs, though relative model-vs-baseline performance should be preserved
- The public tutorial demonstrates CITE-seq (RNA + surface protein) toy data on BMNC, not the SHARE-seq accessibility-to-transcriptome setting, so ATAC-seq-specific evaluation requires the ATAC-seq/peak.tsv folder layout

## Evidence

- [results] Crossmodal generation workflow step: [evidence span withheld — anchor status: not_found]
