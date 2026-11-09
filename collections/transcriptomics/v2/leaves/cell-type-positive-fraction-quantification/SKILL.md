---
name: cell-type-positive-fraction-quantification
description: Use when single-cell AnnData atlases spanning multiple ages (e.g., Tabula
  Muris Senis, 328K cells, 19 tissues, 1-30 months) must be scored for senescence
  and aggregated into per-cell-type positive-cell proportions or old/young ratios.
when_to_use_negative:
- 'Data are scaled: score_hub requires non-scaled data (''Just make sure they are
  not scaled''), so applying the scoring to scaled AnnData matrices is invalid.'
- The goal is single-marker differential expression per se rather than quantifying
  the proportion of cells positive for senescence signatures across cell types and
  ages.
- Input is bulk transcriptomics or lacks cell-type annotations, so positive-cell fractions
  cannot be aggregated per cell type.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides the senescence gene signatures (load_hubs) and the scoring function
    (score_hub) used to binarize and weight per-cell senescence scores
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the mouse and human atlases; scores are stored in adata.obs
- name: scanpy
  role: Preprocessing (normalize_total, log1p) of the AnnData objects prior to scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:ada6a245e7106921bcd8b16b3bec1ebfaf8843f275a3ef86714eede326109ce9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cell-type-positive-fraction-quantification

## Summary

Quantify how the fraction of cells scored positive for cellular senescence signatures (via SenePy hub scoring with binarization and hub-network importance weighting) changes across cell types and age groups in the Tabula Muris Senis mouse atlas and a human aging atlas, recovering marker-specific dynamics such as CDKN2A being dynamic in only ~26% of human and ~32% of mouse cell types.

## When to use

Use when single-cell AnnData atlases spanning multiple ages (e.g., Tabula Muris Senis, 328K cells, 19 tissues, 1-30 months) must be scored for senescence and aggregated into per-cell-type positive-cell proportions or old/young ratios.

## When NOT to use

- Data are scaled: score_hub requires non-scaled data ('Just make sure they are not scaled'), so applying the scoring to scaled AnnData matrices is invalid.
- The goal is single-marker differential expression per se rather than quantifying the proportion of cells positive for senescence signatures across cell types and ages.
- Input is bulk transcriptomics or lacks cell-type annotations, so positive-cell fractions cannot be aggregated per cell type.

## Inputs

- Mouse single-cell atlas (Tabula Muris Senis, 328K cells, 19 tissues, ages 1-30 months) as an AnnData object
- Human single-cell aging atlas as an AnnData object
- SenePy senescence gene signatures loaded via sp.load_hubs(species='Mouse') and sp.load_hubs(species='Human') (181-gene CS marker panel from the hubs)

## Outputs

- Per-cell-type, per-age-group table of proportions of senescence-positive cells (old_young_cs_proportions.csv)
- Old/young positive-cell ratio per cell type and histogram reproducing Fig. 1B
- adata.obs column with per-cell SenePy senescence scores

## How to apply

Load the Tabula Muris Senis mouse atlas and the human single-cell aging atlas as AnnData objects, and load SenePy signatures with sp.load_hubs(species='Mouse'/'Human'). Score every cell with sp.score_hub(adata, signature) using defaults — binarize=True and importance weighting by each gene's number of connections in the hub network — keeping data normalized but not scaled, and store scores in adata.obs. Aggregate the thresholded (binarized) scores per cell type and age group into proportions of senescence-positive cells, then compute old/young positive-cell ratios per cell type. Validate by checking the recovered dynamics: CDKN2A dynamic in ~26% of human and ~32% of mouse cell types, CCL4 broadly dynamic but in only 18 of 50 human cell types, and a histogram of old/young ratios matching Fig. 1B. Write the per-cell-type proportion table to old_young_cs_proportions.csv.

## Related tools

- **SenePy** (Provides the senescence gene signatures (load_hubs) and the scoring function (score_hub) used to binarize and weight per-cell senescence scores) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the mouse and human atlases; scores are stored in adata.obs)
- **scanpy** (Preprocessing (normalize_total, log1p) of the AnnData objects prior to scoring)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Recomputed CDKN2A dynamics match the report: dynamic in ~26% of human and ~32% of mouse cell types.
- CCL4 appears dynamically senescence-associated in 18 of 50 human cell types.
- The histogram of old/young positive-cell ratios matches Fig. 1B.
- The output table old_young_cs_proportions.csv contains one row per cell type with age-group positive fractions.
- adata.obs contains a senescence score column produced with default score_hub settings (binarize=True, importance weighting on).

## Limitations

- Known CS markers are highly cell-type-specific: CDKN2A was dynamic in only 26% of human and 32% of mouse cell types; CCL4, the most broadly dynamic human marker, was dynamic in only 18 of 50 cell types.
- High dropout rate and limited sequencing depth of single-cell transcriptomics limit the use of classical CS markers such as Cdkn2a and Cdkn1a as sole indicators.
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging, and genes downregulated in senescent cells are not accounted for.

## Evidence

- [readme] By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation optional.: "it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] score_hub scoring example from README methods.: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
- [results] Recovered cell-type specificity of known CS markers from the reproduced analysis.: [evidence span withheld — anchor status: not_found]
- [intro] Tabula Muris Senis atlas description grounding the mouse input.: "For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age"
