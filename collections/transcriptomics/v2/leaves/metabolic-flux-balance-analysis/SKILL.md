---
name: metabolic-flux-balance-analysis
description: Use when you have a gene x sample RNA-seq matrix (human gene symbols
  as row names, normalized, non-negative) and need nutrient-aware metabolic flux estimates
  for 13082 reactions, e.g., glucose uptake (HMR_9034), pathway activity, or TME cell-type
  community fluxes.
when_to_use_negative:
- You need absolute quantitative flux measurements rather than relative flux scores
  — use Seahorse XF extracellular flux analysis or LC/MS metabolomics instead
- Input expression data is negative or not normalized (METAFlux requires non-negative,
  normalized gene x sample matrices)
- You need a full in-depth comparison with Compass beyond what the paper's Supplementary
  Note provides, or a specific METAFlux version — no version numbers or changelog
  are documented
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3299
tools:
- name: METAFlux
  role: R package implementing MRAS calculation and FBA-based flux prediction for
    bulk and single-cell RNA-seq
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Runtime environment; package loaded via library(METAFlux)
- name: Seurat
  role: Interface for single-cell Seurat objects (V4) when computing community fluxes
    from scRNA-seq data
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:1fc3735ba76f92ef6aa211aa5438951e4d05d3b2b51ea65a3b0b04bcc1053fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolic-flux-balance-analysis

## Summary

Infer metabolic fluxes from bulk or single-cell RNA-seq data using METAFlux, which combines Human-GEM (13082 reactions, 8378 metabolites) gene-protein-reaction mapping with flux balance analysis under medium constraints. It enables nutrient-aware flux prediction for cancer metabolism studies, NCI-60 benchmarking, and TCGA subgroup/survival analyses.

## When to use

Use when you have a gene x sample RNA-seq matrix (human gene symbols as row names, normalized, non-negative) and need nutrient-aware metabolic flux estimates for 13082 reactions, e.g., glucose uptake (HMR_9034), pathway activity, or TME cell-type community fluxes.

## When NOT to use

- You need absolute quantitative flux measurements rather than relative flux scores — use Seahorse XF extracellular flux analysis or LC/MS metabolomics instead
- Input expression data is negative or not normalized (METAFlux requires non-negative, normalized gene x sample matrices)
- You need a full in-depth comparison with Compass beyond what the paper's Supplementary Note provides, or a specific METAFlux version — no version numbers or changelog are documented

## Inputs

- Gene x sample bulk RNA-seq expression matrix with human gene symbols as row names, normalized and non-negative
- Medium file: cell_medium (44 metabolites, for cell lines) or human_blood (64 metabolites, for human-derived samples)
- Human-GEM metabolic model (13082 reactions, 8378 metabolites) bundled with the METAFlux R package
- Nutrient lookup files mapping metabolites to their 1648 exchange Reaction IDs
- For single-cell: Seurat object with cell-type annotations and cell-type fractions

## Outputs

- Flux scores for 13082 reactions per bulk sample (or 13082 x number of cell-type/cluster + 1648 flux scores per single-cell dataset)
- Optional cbrt-normalized flux matrix
- Per-metabolite exchange fluxes (e.g., glucose uptake via HMR_9034) for downstream comparisons
- Pathway-level activity scores (mean fluxes of reactions associated with each pathway)

## How to apply

Load the expression matrix (gene x sample, human gene symbols as row names, normalized, non-negative) and choose exactly one medium file: cell_medium (44 metabolites) for cell lines or human_blood (64 metabolites) for human-derived samples. Compute single-sample normalized Metabolic Reaction Activity Scores (MRAS) with calculate_reaction_score(), which maps gene expression to reaction activities through Human-GEM GPR rules. Solve the flux balance problem with compute_flux(mras=scores, medium=...) to obtain predicted fluxes for all 13082 reactions, optionally applying cubic-root normalization cbrt(x)=sign(x)*abs(x)^(1/3). Medium constraints are critical: removing them (free uptake/excretion via all 1648 nutrient exchange reactions) drops benchmark accuracy from ρ=0.74 to ρ=0.15 on NCI-60. For single-cell data, average expression per cell type with calculate_avg_exp (bootstrapped per patient) and compute community fluxes with compute_sc_flux using cell-type fractions. Extract metabolite-specific exchange Reaction IDs (e.g., HMR_9034 for glucose uptake) via the nutrient lookup files (1648 exchange reactions).

## Related tools

- **METAFlux** (R package implementing MRAS calculation and FBA-based flux prediction for bulk and single-cell RNA-seq) — https://github.com/KChen-lab/METAFlux
- **R** (Runtime environment; package loaded via library(METAFlux))
- **Seurat** (Interface for single-cell Seurat objects (V4) when computing community fluxes from scRNA-seq data)

## Examples

```
library(METAFlux); scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras=scores, medium=human_blood)
```

## Evaluation signals

- On the NCI-60 benchmark, Spearman correlation between predicted and experimentally measured fluxes (26 metabolites + biomass reaction across 11 cell lines) should reach ρ≈0.74, exceeding ecGEMs (ρ=0.45) in every evaluated cell line
- The no-medium-constraint control should yield a much lower overall Spearman correlation (reported ρ=0.15), confirming medium constraint sensitivity
- Flux output must cover all 13082 Human-GEM reactions per bulk sample
- Extracted exchange Reaction IDs (e.g., HMR_9034 for glucose uptake) must match the nutrient lookup file annotations
- On TCGA LUAD/LUSC, clustering flux profiles should recover a LUSC-like LUAD subgroup with higher glucose uptake, and LUAD1 vs LUSC-like survival log-rank P≈0.0084

## Limitations

- Flux predictions are relative scores, not absolute rates; glucose uptake correlates less consistently than other metabolites on the NCI-60 panel
- The objective function maximizes the generic biomass_human reaction from Human1, which may not capture all metabolic objectives
- No software version numbers for METAFlux, R, or Seurat are documented; the in-depth Compass comparison is only in a Supplementary Note
- Requires choosing exactly one medium file (cell_medium or human_blood); the wrong medium choice materially degrades accuracy

## Evidence

- [methods] MRAS via GPR mapping: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] compute_flux call: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [methods] medium choice: "users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood)"
- [intro] benchmark accuracy: "We evaluate METAFlux prediction accuracy using the matched flux data generated on the NCI-60 cell lines and find a substantial improvement over existing approaches."
- [methods] cbrt normalization: "cbrt <- function(x) {
    sign(x) * abs(x)^(1/3)
}"
- [readme] install method: "METAFlux R package can be easily installed from Github using devtools:

`devtools::install_github('KChen-lab/METAFlux')`"
- [readme] capabilities: [evidence span withheld — anchor status: external_doc]
