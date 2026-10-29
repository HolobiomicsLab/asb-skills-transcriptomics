---
name: age-group-differential-abundance-analysis
description: Use when single-cell AnnData objects annotated with cell type and age
  are available and the goal is to compute, per cell type, the fraction of senescence-score-positive
  cells in young vs old age groups and rank which markers (e.g., CDKN2A, CCL4) are
  most age-dynamic.
when_to_use_negative:
- Input data are bulk transcriptomes or lack per-cell age and cell-type annotations,
  so positive-cell proportions per cell type per age group cannot be computed.
- The data matrix is already scaled (z-scored), which would corrupt the binarized
  score_hub calculation — the README requires data that are 'not scaled'.
- The goal is scoring senescent cells with in vitro-derived signatures (e.g., SenMayo
  alone) rather than the in vivo-derived SenePy hubs, which the paper shows better
  recapitulate in vivo senescence.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: SenePy
  role: Provides load_hubs, merge_hubs, and score_hub to load mouse/human senescence
    signatures and produce binarized per-cell senescence scores whose positive fraction
    is compared across age groups
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the atlas objects; scores are stored in adata.obs for
    per-cell-type/per-age aggregation
- name: scanpy
  role: Preprocessing (normalize_total, log1p) of the AnnData object before scoring;
    data must be normalized but not scaled
- name: Python 3.9
  role: Runtime environment (conda env 'senepy', pip install senepy) for running the
    scoring and aggregation workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:793c31f34ddee81709017c1703acb071454674325686009adf905a17faf0911a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - Python 3.9
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# age-group-differential-abundance-analysis

## Summary

Quantifies how the proportion of senescence-positive cells (SenePy score binarized per cell) changes with organismal age across cell types in the Tabula Muris Senis mouse atlas (328K cells, 19 tissues, ages 1-30 months) and a human single-cell aging atlas, yielding old/young positive-cell ratios that reveal cell-type-specific marker dynamics.

## When to use

Use when single-cell AnnData objects annotated with cell type and age are available and the goal is to compute, per cell type, the fraction of senescence-score-positive cells in young vs old age groups and rank which markers (e.g., CDKN2A, CCL4) are most age-dynamic.

## When NOT to use

- Input data are bulk transcriptomes or lack per-cell age and cell-type annotations, so positive-cell proportions per cell type per age group cannot be computed.
- The data matrix is already scaled (z-scored), which would corrupt the binarized score_hub calculation — the README requires data that are 'not scaled'.
- The goal is scoring senescent cells with in vitro-derived signatures (e.g., SenMayo alone) rather than the in vivo-derived SenePy hubs, which the paper shows better recapitulate in vivo senescence.

## Inputs

- AnnData object of the Tabula Muris Senis mouse atlas (328K cells, 19 tissues, ages 1-30 months) with cell-type and age annotations
- AnnData object of a human single-cell aging atlas (50 cell types) with cell-type and age annotations
- SenePy hub signatures loaded via sp.load_hubs(species='Mouse') and sp.load_hubs(species='Human')
- Known CS marker gene list (e.g., CDKN2A, CCL4, CCL5/CCL8) drawn from the 181-gene CS marker panel

## Outputs

- adata.obs column (e.g., sp_score) with per-cell senescence scores
- Per-cell-type-by-age-group table of senescence-positive cell proportions (old_young_cs_proportions.csv)
- Old/young positive-cell ratio histogram reproducing Fig. 1B
- Ranked list of age-dynamic markers per cell type (e.g., CDKN2A dynamic in ~26% human / ~32% mouse cell types; CCL4 in 18/50 human cell types)

## How to apply

First score every cell with senepy's sp.score_hub(adata, signature) using defaults (binarize=True, importance weighting by hub-network connection count), storing scores in adata.obs; data may be normalized (scanpy normalize_total + log1p) but must not be scaled. Load signatures with sp.load_hubs(species='Mouse'/'Human') and assemble the 181-gene CS marker panel for scoring known markers. Then aggregate per cell type and age group the proportion of senescence-positive cells (thresholded score) in both atlases, compute old/young positive-cell ratios per cell type, and plot the ratio histogram (Fig. 1B) plus a per-cell-type proportion table (old_young_cs_proportions.csv). Rationale: the binarized score turns continuous per-cell scores into a positive/negative call, so differential abundance reduces to comparing positive-cell fractions across age groups.

## Related tools

- **SenePy** (Provides load_hubs, merge_hubs, and score_hub to load mouse/human senescence signatures and produce binarized per-cell senescence scores whose positive fraction is compared across age groups) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the atlas objects; scores are stored in adata.obs for per-cell-type/per-age aggregation)
- **scanpy** (Preprocessing (normalize_total, log1p) of the AnnData object before scoring; data must be normalized but not scaled)
- **Python 3.9** (Runtime environment (conda env 'senepy', pip install senepy) for running the scoring and aggregation workflow)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Recovered marker dynamics match the published values: CDKN2A dynamic in ~26% of human and ~32% of mouse cell types; CCL4 dynamic in 18 of 50 human cell types.
- The old/young positive-cell ratio histogram matches Fig. 1B of the paper.
- The per-cell-type proportion table (old_young_cs_proportions.csv) is complete over all cell types x age groups with proportions in [0, 1].
- score_hub was called with default binarize=True and importance=True on data that are normalized (normalize_total + log1p) but not scaled.
- Per-age-group cell counts are non-zero and the thresholded scores define a valid binary positive/negative call per cell.

## Limitations

- Known CS marker dynamics are highly cell-type-specific: CDKN2A was dynamic in only a minority of cell types (26% human, 32% mouse), and CCL4 in only 18/50 human cell types, so a marker absent in one cell type is not evidence of no senescence.
- High dropout rate and limited sequencing depth of single-cell transcriptomics limit use of classical CS markers such as Cdkn2a and Cdkn1a as sole indicators; transient p16 expression combined with gene dropout yields low Cdkn2a detectability.
- No universal CS signature gene panel exists due to cell-specific heterogeneity of CS; cell-type-specific hubs should be preferred over a single universal list.
- The SenePy methodology discounts CS genes that overlap with the transcriptional shift of organismal aging and does not account for genes downregulated in senescent cells (no negative markers).

## Evidence

- [results] CDKN2A dynamic in only 26% of human and 32% of mouse cell types, while CCL4 was the most broadly dynamic human marker yet only in 18 of 50 cell types: [evidence span withheld — anchor status: not_found]
- [intro] For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age: "For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age"
- [readme] By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation optional.: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data"
- [discussion] the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators: "the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators"
- [readme] Just make sure they are not scaled: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
