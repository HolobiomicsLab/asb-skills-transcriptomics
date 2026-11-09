---
name: cross-species-aging-atlas-analysis
description: Use when you need to reproduce or extend cell-type-by-age analyses of
  cellular senescence in single-cell atlases spanning multiple ages/species, e.g.
  scoring cells with SenePy signatures and comparing old vs young marker-positive
  proportions per cell type.
when_to_use_negative:
- The input is already a scored/feature table of senescence positivity rather than
  raw or normalized single-cell counts in AnnData — re-scoring would be redundant.
- The data are scaled (z-scored) — score_hub explicitly requires non-scaled data for
  correct binarized scoring.
- The goal is differential aging expression rather than senescence-marker proportion
  dynamics; SenePy signatures discount genes overlapping the general transcriptional
  shift of aging and do not capture downregulated genes.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Provides the cell-type-specific senescence hub signatures (load_hubs, merge_hubs)
    and the score_hub function used to binarize and importance-weight per-cell senescence
    scores.
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the Tabula Muris Senis and human aging atlases; scores
    are written to adata.obs.
- name: scanpy
  role: Preprocessing of AnnData (normalize_total, log1p) prior to scoring; score_hub
    can mimic sc.tl.score_genes when binarize=False and importance=False.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:983bfa59d1973beededb16cf4aaf28a0c9ccdae153d835d20c6c8bbd7a7791d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-species-aging-atlas-analysis

## Summary

Quantifies how the proportion of senescence-positive cells changes with age across cell types in the Tabula Muris Senis mouse atlas (328K cells, 19 tissues, 1–30 months) and a human single-cell aging atlas, using SenePy cell-type-specific senescence hub signatures scored per cell and aggregated into old/young positive-cell ratios.

## When to use

Use when you need to reproduce or extend cell-type-by-age analyses of cellular senescence in single-cell atlases spanning multiple ages/species, e.g. scoring cells with SenePy signatures and comparing old vs young marker-positive proportions per cell type.

## When NOT to use

- The input is already a scored/feature table of senescence positivity rather than raw or normalized single-cell counts in AnnData — re-scoring would be redundant.
- The data are scaled (z-scored) — score_hub explicitly requires non-scaled data for correct binarized scoring.
- The goal is differential aging expression rather than senescence-marker proportion dynamics; SenePy signatures discount genes overlapping the general transcriptional shift of aging and do not capture downregulated genes.

## Inputs

- Tabula Muris Senis mouse atlas as AnnData (.h5ad), 328K cells, 19 tissues, ages 1–30 months
- Human single-cell aging atlas as AnnData (.h5ad)
- SenePy mouse and human senescence hub signatures (75 mouse / 65 human signatures via sp.load_hubs)
- Senescence marker gene list (e.g. CDKN2A, CCL4, Ccl5/Ccl8) for targeted dynamics checks

## Outputs

- adata.obs columns with per-cell senescence scores (sp_score)
- Per-cell-type, per-age-group proportions of senescence-positive cells for mouse and human
- Old/young positive-cell ratio table exported as old_young_cs_proportions.csv
- Histogram of old/young positive-cell ratios reproducing Fig. 1B

## How to apply

Load the Tabula Muris Senis AnnData (328K cells from 19 tissues, ages 1–30 months) and a human single-cell aging atlas, then retrieve weighted senescence signatures with senepy's sp.load_hubs(species='Mouse') and sp.load_hubs(species='Human') and merge hubs into the 181-gene CS marker panel via merge_hubs. Score every cell with sp.score_hub(adata, signature) using default settings — binarize=True and importance weighting by hub-network connection count — ensuring data are normalized/log1p'd but NOT scaled, and store results in adata.obs. Aggregate the thresholded positive-cell proportion per cell type and age group, then compute old/young positive-cell ratios per cell type for mouse and human separately. Rationale: because classical markers like Cdkn2a suffer from high dropout in single-cell data, the signature-based scoring and binarization makes normalization/transformation optional and captures cell-type-specific CS programs. Validate by checking that recomputed dynamics match the reported CDKN2A (dynamic in ~26% of human, ~32% of mouse cell types) and CCL4 (18/50 human cell types) figures, and that the old/young ratio histogram reproduces Fig. 1B before exporting old_young_cs_proportions.csv.

## Related tools

- **SenePy** (Provides the cell-type-specific senescence hub signatures (load_hubs, merge_hubs) and the score_hub function used to binarize and importance-weight per-cell senescence scores.) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the Tabula Muris Senis and human aging atlases; scores are written to adata.obs.)
- **scanpy** (Preprocessing of AnnData (normalize_total, log1p) prior to scoring; score_hub can mimic sc.tl.score_genes when binarize=False and importance=False.)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Recomputed CDKN2A dynamics match reported values: dynamic in ~26% of human and ~32% of mouse cell types.
- CCL4 recovery matches: the most broadly dynamic human marker, dynamic in 18/50 human cell types.
- Distribution of old/young senescence-positive-cell ratios reproduces Fig. 1B.
- Per-cell-type proportion table old_young_cs_proportions.csv contains all cell types × age groups from both atlases with plausible proportions in [0, 1].
- Verification that adata was normalized (sc.pp.normalize_total + sc.pp.log1p) but not scaled before score_hub was applied.

## Limitations

- High dropout rate and limited sequencing depth of single-cell transcriptomics limit detectability of classical CS markers such as Cdkn2a and Cdkn1a as sole indicators.
- Known CS marker dynamics are highly cell-type-specific: CDKN2A was dynamic in only 26% of human and 32% of mouse cell types, CCL4 in 18/50 — so individual markers cannot substitute for signature-level scoring.
- The methodology discounts CS genes that overlap the transcriptional shift of organismal aging and does not account for genes downregulated in senescent cells (no negative markers).

## Evidence

- [intro] Tabula Muris Senis resource description: "For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age"
- [readme] score_hub defaults: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation"
- [readme] normalization requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [discussion] dropout limitation: "the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators"
- [methods] hub database scope: "Database of 75 **mouse** and 65 **human** signatures of cellular senescence from 97 unique cell types"
