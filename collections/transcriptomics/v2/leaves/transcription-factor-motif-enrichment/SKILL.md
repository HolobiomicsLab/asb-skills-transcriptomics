---
name: transcription-factor-motif-enrichment
description: 'Use when a Mowgli factor''s H(atac) matrix rows must be biologically
  annotated: top-weighted peaks per factor are tested for TF motif over-representation
  to name the factor''s putative regulator.'
when_to_use_negative:
- Input is not a Mowgli factor decomposition with an ATAC modality — there are no
  factor-specific peaks to test
- Peak annotation/binarization metadata or a suitable background peak set is unavailable,
  since FindMotifs requires background peaks for valid over-representation testing
- The goal is annotating RNA-side factors; use gene set enrichment (gProfiler via
  Scanpy's enrich) on the 150 top genes instead
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Signac
  role: Performs Motif Enrichment Analysis on the 100 top peaks per factor using the
    union of top peaks across factors as background
- name: Mowgli
  role: Produces the factor H(atac) matrix whose top-weighted peaks are the input
    to motif enrichment
  repo: https://github.com/cantinilab/mowgli
- name: Regulatory Circuits
  role: Source of TF-gene links used to star biologically interesting TFs and corroborate
    motif-based factor assignments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:81c14055f3efd59cb4efda4e34af5ee550d7e2238420219ef4502dc9f2cf9364
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Azimuth
  - Human Protein Atlas
  - Mowgli
  - Python
  - Regulatory Circuits
  - Scanpy
  - Signac
  - gProfiler
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# transcription-factor-motif-enrichment

## Summary

Interprets factors from Mowgli's multi-omics decomposition by testing whether their top-weighted ATAC peaks are enriched for transcription factor binding motifs, using Signac's Motif Enrichment Analysis against the JASPAR2022 database to assign biological identity (TFs) to latent factors.

## When to use

Use when a Mowgli factor's H(atac) matrix rows must be biologically annotated: top-weighted peaks per factor are tested for TF motif over-representation to name the factor's putative regulator.

## When NOT to use

- Input is not a Mowgli factor decomposition with an ATAC modality — there are no factor-specific peaks to test
- Peak annotation/binarization metadata or a suitable background peak set is unavailable, since FindMotifs requires background peaks for valid over-representation testing
- The goal is annotating RNA-side factors; use gene set enrichment (gProfiler via Scanpy's enrich) on the 150 top genes instead

## Inputs

- Rows of the Mowgli H(atac) factor matrix normalized to 1, from a TEA-seq (or similar) multi-omics analysis
- Top 100 peaks per factor by weight
- Background peak set: union of top peaks across all factors
- JASPAR2022 motif database

## Outputs

- Per-factor over-represented TF binding motifs with enrichment p-values
- Factor-to-TF assignments used in the biological interpretation panel (Fig. 5-style output)

## How to apply

For each Mowgli factor, take the rows of H(atac) normalized to sum to 1 and select the 100 top peaks as input to Signac's FindMotifs. Use the union of the top peaks across all factors as the background peak set, so enrichment reflects factor-specific rather than globally accessible motifs. Run the motif enrichment against the JASPAR2022 motif database to identify which TF binding motifs are over-represented in each factor's peaks. Cross-reference the resulting TFs with gene set enrichment of the factor's 150 top genes and TF-gene links from the Regulatory Circuits networks to star biologically interesting genes and confirm factor–TF–cell-type assignments.

## Related tools

- **Signac** (Performs Motif Enrichment Analysis on the 100 top peaks per factor using the union of top peaks across factors as background)
- **Mowgli** (Produces the factor H(atac) matrix whose top-weighted peaks are the input to motif enrichment) — https://github.com/cantinilab/mowgli
- **Regulatory Circuits** (Source of TF-gene links used to star biologically interesting TFs and corroborate motif-based factor assignments)

## Examples

```
motifs <- FindMotifs(object = atac.seurat, features = top100_peaks_per_factor, background = union_top_peaks_all_factors)  # JASPAR2022 motifs
```

## Evaluation signals

- Exactly 100 top peaks per factor are submitted (rows of H(atac) normalized to 1), matching the paper's protocol
- The background peak set equals the union of top peaks across all factors, not all genome-wide peaks
- Enrichment is computed against JASPAR2022; results should name TFs consistent with the factor's assigned immune cell type (e.g., NK/CD8/CD4 T-cell factors) from the specificity-score annotation
- Motif-derived TF assignments should be concordant with GO/Azimuth/Immune SigDB gene set enrichment (Bonferroni-adjusted p < 0.05) on the same factor's 150 top genes

## Limitations

- The Regulatory Circuits website (http://ww1.regulatorycircuits.org/) used for TF-gene links was down at the time of writing; data had to be recovered from a mirror
- Motif enrichment is only meaningful for factors with an ATAC component; RNA/ADT-only factors yield no peaks to test
- Factor number (k) in Mowgli is not straightforward to define, so the set of factors to annotate may need to be selected from several tested k values

## Evidence

- [methods] Signac used for motif enrichment on top peaks: "Signac85 was used to perform Motif Enrichment Analysis, using the JASPAR2022 motif database86."
- [methods] Top 100 peaks per factor as motif enrichment input: [evidence span withheld — anchor status: not_found]
- [supplementary] Regulatory Circuits website down; data recovered from mirror: "At the time of writing, the Regulatory Circuits website http://ww1.regulatorycircuits.org/ is down."
- [methods] Mowgli decomposes each modality into H and W embeddings: [evidence span withheld — anchor status: not_found]
