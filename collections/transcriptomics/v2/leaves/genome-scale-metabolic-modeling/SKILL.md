---
name: genome-scale-metabolic-modeling
description: Use when transcriptomic data (bulk RNA-seq matrix, Seurat object, or
  cell-cluster expression profiles) must be converted into reaction-level flux distributions
  under a defined nutrient medium, e.g. nutrient-aware cancer metabolism analysis
  of TCGA or single-cell TME data.
when_to_use_negative:
- When the goal is direct measurement of extracellular fluxes or metabolite concentrations
  — use Seahorse XF analyzer, LC/MS, or MALDI-MS instead of transcriptome-inferred
  fluxes
- When input is not gene expression but an already-computed flux or reaction-score
  table — skip MRAS/GPR mapping and go straight to flux solving
- When nutrient availability is unknown or the experiment does not fit binary nutrient
  presence/absence conditions — medium definition is required for uptake-restricted
  bounds
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: METAFlux
  role: R package implementing GPR-based MRAS calculation and QP flux balance analysis
    on Human-GEM
  repo: https://github.com/KChen-lab/METAFlux
- name: osqp
  role: Convex quadratic-programming solver for the parsimonious FBA objective over
    all 13,082 reactions
- name: R
  role: Runtime environment for the METAFlux package and its compute_flux / calculate_reaction_score
    functions
- name: Seurat
  role: Interfaces single-cell data (Seurat V4 objects) for per-patient bootstrapped
    average expression and community flux computation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:1e9e0dbcddabd24426389f48d0191ae04e9f8be116fb5e4543f3531218efd160
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  - osqp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# genome-scale-metabolic-modeling

## Summary

Predict sample- and cell-type-specific metabolic fluxes by constraining a genome-scale metabolic model (Human-GEM, 13,082 reactions / 8,378 metabolites) with gene-expression-derived reaction activity scores and solving a parsimonious convex-QP FBA problem with osqp.

## When to use

Use when transcriptomic data (bulk RNA-seq matrix, Seurat object, or cell-cluster expression profiles) must be converted into reaction-level flux distributions under a defined nutrient medium, e.g. nutrient-aware cancer metabolism analysis of TCGA or single-cell TME data.

## When NOT to use

- When the goal is direct measurement of extracellular fluxes or metabolite concentrations — use Seahorse XF analyzer, LC/MS, or MALDI-MS instead of transcriptome-inferred fluxes
- When input is not gene expression but an already-computed flux or reaction-score table — skip MRAS/GPR mapping and go straight to flux solving
- When nutrient availability is unknown or the experiment does not fit binary nutrient presence/absence conditions — medium definition is required for uptake-restricted bounds

## Inputs

- Gene expression matrix (gene x sample, human gene symbols as row names, normalized, non-negative), e.g. bulk_test_example
- Human-GEM genome-scale metabolic model (13,082 reactions, 8,378 metabolites, with GPR associations, reversibility, compartments, SUBSYSTEM labels)
- Medium file defining exchange-reaction bounds: human_blood or cell_medium, plus nutrient lookup files (1648 exchange reactions)
- For single-cell mode: Seurat object with cell-type annotations and cell-type fractions

## Outputs

- MRAS matrix (13,082 reactions x samples) keyed by Human-GEM Reaction IDs
- Per-sample flux vector over all 13,082 Human-GEM reactions (fluxes.csv)
- For single-cell data: (13,082 x number of cell-type/cluster + 1648) reaction flux scores
- Optional pathway-level activities (mean fluxes of reactions in each SUBSYSTEM pathway)

## How to apply

Load the gene expression matrix (gene x sample, human gene symbols as row names, no negative entries) and the Human-GEM model. Map gene expression to reaction activity via GPR rules — minimum across subunits for AND logic, summation for OR logic, with a promiscuity adjustment dividing each gene's expression by its number of reactions — then per-sample normalize to obtain MRAS. Set E-Flux-style bounds (upper bound = normalized MRAS, lower bound = -MRAS for reversible reactions, 0 otherwise) and restrict uptake to metabolites in the chosen medium file (human_blood for human-derived samples, cell_medium for cell lines, 1648 exchange reactions in the nutrient lookup files). Solve the convex QP that maximizes the biomass objective while minimizing the sum of squared fluxes (min 1/2 v^T v - a C^T v, weight a=10000, S v ≈ 0 within equality tolerance) using the osqp solver, then optionally apply cubic-root normalization sign(x)*|x|^(1/3). For single-cell data, compute community fluxes by supplying cell-type fractions per patient (optionally via stratified bootstrapped average expression profiles).

## Related tools

- **METAFlux** (R package implementing GPR-based MRAS calculation and QP flux balance analysis on Human-GEM) — https://github.com/KChen-lab/METAFlux
- **osqp** (Convex quadratic-programming solver for the parsimonious FBA objective over all 13,082 reactions)
- **R** (Runtime environment for the METAFlux package and its compute_flux / calculate_reaction_score functions)
- **Seurat** (Interfaces single-cell data (Seurat V4 objects) for per-patient bootstrapped average expression and community flux computation)

## Examples

```
scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras=scores, medium=human_blood)
```

## Evaluation signals

- Output flux vector contains exactly one flux value per reaction for all 13,082 Human-GEM Reaction IDs, with no missing values
- Exchange reactions show positive values for release and negative values for uptake, and uptake is only possible for metabolites present in the chosen medium file
- Result is reproducible against the reference compute_flux(mras=scores, medium=human_blood) call on the bulk_test_example toy dataset
- MRAS matrix has 13,082 reaction rows keyed by Human-GEM Reaction IDs, one column per sample, and no missing or negative scores
- Predicted fluxes should be consistent with matched experimental measurements where available (e.g. NCI-60 uptake/release rates, Seahorse extracellular flux on the Raji-NK co-culture)

## Limitations

- The biomass objective is the generic biomass_human reaction from Human1, so fluxes reflect biomass-maximizing parsimony rather than measured fluxes; glucose uptake predicted for the NCI-60 panel did not correlate as consistently as other metabolites
- Nutrient assumptions depend on the single medium file chosen (cell_medium vs human_blood); only one medium should be used per run and it must be customized by deleting/adding nutrient rows
- No software version numbers for METAFlux, R, or Seurat are stated, and the in-depth comparison with Compass is only in a Supplementary Note not covered here

## Evidence

- [methods] Human-GEM model basis: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] QP objective follows parsimonious FBA: [evidence span withheld — anchor status: not_found]
- [discussion] Biomass objective source: "current objective function of METAFlux maximizes the flux of generic biomass reaction (biomass_human from Human1)"
- [readme] README capability statement: [evidence span withheld — anchor status: external_doc]
- [intro] Nutrient-aware fluxes: "METAFlux is capable of characterizing the entire metabolic circuits and output non-degenerative fluxes using cancer gene expression data in a nutrient-aware manner."
