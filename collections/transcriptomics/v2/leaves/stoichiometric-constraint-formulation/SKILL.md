---
name: stoichiometric-constraint-formulation
description: Use when bulk RNA-seq samples require sample-specific metabolic flux
  estimates over a genome-scale model, and gene expression (MRAS) plus a nutrient
  medium must be translated into E-Flux style bounds for a QP-constrained FBA solve.
when_to_use_negative:
- Input is single-cell RNA-seq data requiring community/tumor-microenvironment-aware
  fluxes — use compute_sc_flux with cell-type fractions instead of the bulk QP formulation
- Only pathway-level reaction activity scores (MRAS) are needed and no steady-state
  stoichiometric constraint is required — calculate_reaction_score alone suffices
- No nutrient medium context is available or the analysis needs unlimited uptake conditions,
  since this formulation restricts uptake to medium-defined exchange reactions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: METAFlux
  role: R package whose compute_flux function implements the convex QP FBA formulation
    over Human-GEM with medium-defined exchange bounds
  repo: https://github.com/KChen-lab/METAFlux
- name: osqp
  role: R solver used to solve the convex quadratic program (min 1/2 v^T v - a C^T
    v, a=10000) under S v ≈ 0 constraints
- name: R
  role: Host language for the METAFlux package and the compute_flux call
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:462121fd12766b11e39ba723324cf16d76ae8b47a7d0f71a8782a3b147bd896f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - osqp
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# stoichiometric-constraint-formulation

## Summary

Formulate and solve a convex quadratic-programming FBA problem over the Human-GEM stoichiometric matrix (13,082 reactions, 8,378 metabolites) to estimate per-sample metabolic fluxes, as implemented in METAFlux's compute_flux step. The objective simultaneously maximizes the biomass_human reaction and minimizes the sum of squared fluxes (parsimonious-FBA style), subject to MRAS-derived and medium-defined exchange bounds.

## When to use

Use when bulk RNA-seq samples require sample-specific metabolic flux estimates over a genome-scale model, and gene expression (MRAS) plus a nutrient medium must be translated into E-Flux style bounds for a QP-constrained FBA solve.

## When NOT to use

- Input is single-cell RNA-seq data requiring community/tumor-microenvironment-aware fluxes — use compute_sc_flux with cell-type fractions instead of the bulk QP formulation
- Only pathway-level reaction activity scores (MRAS) are needed and no steady-state stoichiometric constraint is required — calculate_reaction_score alone suffices
- No nutrient medium context is available or the analysis needs unlimited uptake conditions, since this formulation restricts uptake to medium-defined exchange reactions

## Inputs

- Single-sample normalized MRAS vector per bulk sample (from calculate_reaction_score on a gene x sample matrix with human gene symbols as row names)
- Medium file (human_blood for human-derived samples, or cell_medium for cell line models)
- Human-GEM metabolic model (13,082 reactions, 8,378 metabolites) with stoichiometric matrix S and biomass_human objective from Human1

## Outputs

- Per-reaction flux vector of 13,082 Human-GEM Reaction IDs per bulk sample (e.g. written to fluxes.csv)
- Optionally cubic-root-normalized flux scores (sign(x)*|x|^(1/3))

## How to apply

Load the single-sample normalized MRAS (computed via calculate_reaction_score from a gene x sample expression matrix) and the chosen medium file (human_blood for human-derived samples or cell_medium for cell lines). Set E-Flux style bounds: upper bound = normalized MRAS per reaction, lower bound = -MRAS for reversible reactions and 0 for irreversible ones. Build the convex QP objective min 1/2 v^T v - a C^T v with weight a=10000, constrained by S v ≈ 0 with equality tolerance and by medium-defined exchange-reaction bounds so only medium metabolites can be taken up. Solve with the osqp solver to obtain a parsimonious flux distribution over all 13,082 Human-GEM reactions, then optionally apply cubic-root normalization sign(x)*|x|^(1/3) and write the per-reaction flux vector indexed by Human-GEM Reaction ID.

## Related tools

- **METAFlux** (R package whose compute_flux function implements the convex QP FBA formulation over Human-GEM with medium-defined exchange bounds) — https://github.com/KChen-lab/METAFlux
- **osqp** (R solver used to solve the convex quadratic program (min 1/2 v^T v - a C^T v, a=10000) under S v ≈ 0 constraints)
- **R** (Host language for the METAFlux package and the compute_flux call)

## Examples

```
flux <- compute_flux(mras=scores, medium = human_blood)
```

## Evaluation signals

- Output contains exactly one flux value for each of the 13,082 Human-GEM reactions
- Exchange reactions show positive flux values for metabolite release and negative values for uptake
- Solved fluxes satisfy S v ≈ 0 within the equality tolerance of the QP (steady-state mass balance holds)
- Flux vector is reproducible against the reference compute_flux(mras=scores, medium=human_blood) result on the bulk_test_example toy dataset
- Only medium-defined exchange reactions (e.g. the 1648 exchange reactions in the nutrient lookup files) permit nonzero metabolite uptake

## Limitations

- The current objective function maximizes the generic biomass_human reaction from Human1; fluxes depend on this choice of objective
- Uptake is restricted to medium-defined exchange reactions, so metabolites absent from the chosen medium (human_blood or cell_medium) cannot be taken up
- No software version number for METAFlux is stated in the provided text; only nutrient presence vs absence (binary conditions) is supported for medium customization

## Evidence

- [methods] Human-GEM model with 13,082 reactions as the constraint basis: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] compute_flux is the concrete call that produces the 13,082 reaction fluxes: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [methods] Optional cubic-root normalization applied to solved fluxes: "optional: flux scores cubic root normalization"
- [readme] osqp is the QP solver dependency for the optimization: [evidence span withheld — anchor status: external_doc]
- [discussion]  biomass_human objective from Human1 defines the maximization term: "current objective function of METAFlux maximizes the flux of generic biomass reaction (biomass_human from Human1)"
