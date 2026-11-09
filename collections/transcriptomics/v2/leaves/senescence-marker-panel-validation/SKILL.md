---
name: senescence-marker-panel-validation
description: Use when a workflow must reproduce or verify age-dependent CS marker
  behavior across cell types in Tabula Muris Senis or human aging atlases using senepy
  hub scoring.
when_to_use_negative:
- Input data are scaled, which the article explicitly warns against — score_hub requires
  data that are 'not scaled'
- The goal is de novo discovery of senescence signatures rather than validating dynamics
  of known CS markers (use the signature-generation algorithm instead)
- The dataset is bulk RNA-seq or spatial transcriptomics; score_hub is designed for
  single-cell gene expression AnnData objects
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs/get_genes/score_hub for assembling the CS marker panel
    and binarizing per-cell senescence scores
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the atlas data; scores are stored in adata.obs
- name: scanpy
  role: Preprocessing (normalize_total, log1p) of AnnData before scoring
- name: Python
  role: Execution environment for the senepy/scanpy workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:41a15d091cd7f6d1c0cd1d7220c31f69b90ec0a75169b5e22b691591bed93fdb
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# senescence-marker-panel-validation

## Summary

Validates known cellular senescence (CS) marker dynamics (e.g., CDKN2A, CCL4) by scoring single-cell atlases with SenePy cell-type-specific signatures and comparing recomputed old/young senescence-positive cell proportions against published figures.

## When to use

Use when a workflow must reproduce or verify age-dependent CS marker behavior across cell types in Tabula Muris Senis or human aging atlases using senepy hub scoring.

## When NOT to use

- Input data are scaled, which the article explicitly warns against — score_hub requires data that are 'not scaled'
- The goal is de novo discovery of senescence signatures rather than validating dynamics of known CS markers (use the signature-generation algorithm instead)
- The dataset is bulk RNA-seq or spatial transcriptomics; score_hub is designed for single-cell gene expression AnnData objects

## Inputs

- Tabula Muris Senis mouse atlas as AnnData (.h5ad), 328K cells, 19 tissues, ages 1-30 months
- Human single-cell aging atlas as AnnData
- senepy hub objects for Mouse and Human (cell-type-specific senescence signatures, ~180-gene CS marker panel)

## Outputs

- Per-cell-type table of old/young senescence-positive cell proportions (old_young_cs_proportions.csv)
- Histogram of old/young positive-cell ratios reproducing Fig. 1B
- Recovered per-marker dynamics summary (e.g., CDKN2A ~26% human / ~32% mouse cell types; CCL4 18/50 human cell types)

## How to apply

Load the Tabula Muris Senis mouse atlas (328K cells, 19 tissues, ages 1-30 months) and the human single-cell aging atlas as AnnData objects. Initialize senepy hub objects with sp.load_hubs(species='Mouse') and sp.load_hubs(species='Human') and assemble the ~180-gene CS marker panel; scores should be computed with sp.score_hub(adata, hubs.hubs[...]) under default settings (binarize=True, importance weighting by hub-network connection count), ensuring the AnnData are normalized/log1p'd but not scaled. Aggregate per cell type and age group the proportion of senescence-positive cells (binarized threshold), then compute old/young positive-cell ratios per cell type. Validate by checking that recomputed marker dynamics match reported values: CDKN2A dynamic in ~26% of human and ~32% of mouse cell types, and CCL4 dynamic in 18 of 50 human cell types, and that the old/young ratio distribution reproduces Fig. 1B. Write the per-cell-type proportion table to old_young_cs_proportions.csv for auditability.

## Related tools

- **SenePy** (Provides load_hubs/get_genes/score_hub for assembling the CS marker panel and binarizing per-cell senescence scores) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the atlas data; scores are stored in adata.obs)
- **scanpy** (Preprocessing (normalize_total, log1p) of AnnData before scoring)
- **Python** (Execution environment for the senepy/scanpy workflow)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Recomputed table matches reported marker dynamics: CDKN2A dynamic in ~26% of human and ~32% of mouse cell types
- CCL4 identified as most broadly dynamic human marker, dynamic in 18 of 50 human cell types
- Histogram of old/young positive-cell ratios visually matches Fig. 1B
- AnnData were normalized and log1p-transformed but not scaled before score_hub
- old_young_cs_proportions.csv contains one row per cell type with old/young positive-cell ratios and validates against the published values

## Limitations

- Known CS markers are highly cell-type-specific: CDKN2A was dynamic in only a minority of cell types (26% human, 32% mouse), partly because transient p16 expression combined with gene dropout yields low Cdkn2a detectability in single-cell data
- SenePy does not account for genes that are downregulated in senescent cells (no negative markers)
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging
- Cell-specific heterogeneity of CS makes a universal CS signature gene panel challenging

## Evidence

- [readme] score_hub example with default settings: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
- [readme] default binarization and importance weighting: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation"
- [readme] n_sen counts prior known senescence markers in hubs: "n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."
- [intro] Tabula Muris Senis dataset scope: "For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age"
- [methods] SenePy integrates with AnnData and scanpy: "Integrates with AnnData and scanpy"
- [discussion] Cdkn2a detectability limitation: "This likely arose from the transient expression of p16 earlier on in the CS program in combination with gene dropout inherent in single-cell sequencing."
