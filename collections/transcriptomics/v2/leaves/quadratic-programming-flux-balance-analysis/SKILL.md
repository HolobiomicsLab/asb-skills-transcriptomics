---
name: quadratic-programming-flux-balance-analysis
description: 'Use when estimating per-sample metabolic reaction fluxes from bulk RNA-seq
  gene expression: you have MRAS reaction scores, a chosen medium file (human_blood
  or cell_medium), and need parsimonious fluxes across all 13,082 Human-GEM reactions.'
when_to_use_negative:
- Input is single-cell data with multiple cell types/clusters — use the community-aware
  compute_sc_flux with cell-type fractions instead of the bulk QP
- Goal is computing reaction activity scores from raw gene expression rather than
  fluxes — calculate_reaction_score handles that step; the QP requires MRAS as input
- The analysis needs uptake/release queries only — search the nutrient lookup file
  for exchange Reaction IDs rather than re-solving the full QP
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3170
tools:
- name: METAFlux
  role: R package providing compute_flux, which formulates and solves the convex QP
    over Human-GEM with MRAS-derived bounds
  repo: https://github.com/KChen-lab/METAFlux
- name: osqp
  role: R optimization solver used to solve the convex quadratic program for the flux
    distribution
- name: R
  role: Runtime environment; METAFlux is loaded via library(METAFlux)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:e2eb63625479fc1b48f5d5c6a3155f5954fe7ac982dbd944db9c69cf8a19787c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - osqp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# quadratic-programming-flux-balance-analysis

## Summary

METAFlux's bulk-sample flux estimation step: a convex quadratic program over the Human-GEM stoichiometric model (13,082 reactions, 8,378 metabolites) that maximizes the biomass_human objective while minimizing the sum of squared fluxes, with E-Flux-style bounds set from normalized MRAS and medium-defined exchange constraints.

## When to use

Use when estimating per-sample metabolic reaction fluxes from bulk RNA-seq gene expression: you have MRAS reaction scores, a chosen medium file (human_blood or cell_medium), and need parsimonious fluxes across all 13,082 Human-GEM reactions.

## When NOT to use

- Input is single-cell data with multiple cell types/clusters — use the community-aware compute_sc_flux with cell-type fractions instead of the bulk QP
- Goal is computing reaction activity scores from raw gene expression rather than fluxes — calculate_reaction_score handles that step; the QP requires MRAS as input
- The analysis needs uptake/release queries only — search the nutrient lookup file for exchange Reaction IDs rather than re-solving the full QP

## Inputs

- Normalized MRAS reaction activity scores per sample (output of calculate_reaction_score)
- Human-GEM genome-scale metabolic model (13,082 reactions, 8,378 metabolites)
- Medium file (human_blood for human-derived samples, or cell_medium for cell lines)
- Nutrient lookup files (1648 exchange reactions) for mapping exchange Reaction IDs

## Outputs

- Per-reaction flux vector over all 13,082 Human-GEM reactions, indexed by Reaction ID (e.g., fluxes.csv)
- Optional cubic-root-normalized flux scores

## How to apply

First compute a single-sample normalized MRAS from the gene-by-sample expression matrix using calculate_reaction_score, which applies GPR rules. Set flux bounds E-Flux style: upper bound equal to the normalized MRAS per reaction, lower bound equal to -MRAS for reversible reactions and 0 otherwise. Build the convex QP objective min 1/2 v^T v - a C^T v with weight a=10000, constrained by S v ≈ 0 (equality tolerance) and restricted by medium-defined exchange-reaction bounds so only metabolites in the chosen medium (e.g., human_blood, whose nutrient lookup files contain 1648 exchange reactions) can be taken up. Solve with the osqp R package to obtain the parsimonious flux distribution; optionally apply cubic-root normalization, sign(x)*|x|^(1/3). The rationale is parsimonious-FBA: simultaneously optimizing biomass and minimal total squared flux avoids degenerate flux solutions while the biomass_human reaction from Human1 anchors the objective.

## Related tools

- **METAFlux** (R package providing compute_flux, which formulates and solves the convex QP over Human-GEM with MRAS-derived bounds) — https://github.com/KChen-lab/METAFlux
- **osqp** (R optimization solver used to solve the convex quadratic program for the flux distribution)
- **R** (Runtime environment; METAFlux is loaded via library(METAFlux))

## Examples

```
flux <- compute_flux(mras = scores, medium = human_blood)
```

## Evaluation signals

- Output contains exactly one flux value for each of the 13,082 Human-GEM reactions
- Exchange reactions show negative values for uptake and positive values for release
- Only medium metabolites can be taken up: uptake fluxes must be zero for exchange reactions excluded by the medium-defined bounds
- Flux vector is reproducible against the compute_flux(mras=scores, medium=human_blood) reference result on the bulk_test_example toy dataset
- Applying the optional cbrt normalization must preserve sign: sign(x)*|x|^(1/3)

## Limitations

- The current objective function maximizes flux of the generic biomass reaction (biomass_human from Human1), so results depend on that biomass definition
- Predicted glucose uptake does not correlate as consistently as other metabolites on the NCI-60 panel
- No version number for METAFlux, R, or osqp is stated; users must install osqp separately if installation issues occur
- Users must choose exactly one medium file (either cell_medium or human_blood); medium choice determines which uptakes are feasible

## Evidence

- [methods] Human-GEM model scale: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] flux computation step: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [readme] osqp dependency: [evidence span withheld — anchor status: external_doc]
- [readme] FBA capability: [evidence span withheld — anchor status: external_doc]
- [discussion] biomass objective grounding: "current objective function of METAFlux maximizes the flux of generic biomass reaction (biomass_human from Human1)"
