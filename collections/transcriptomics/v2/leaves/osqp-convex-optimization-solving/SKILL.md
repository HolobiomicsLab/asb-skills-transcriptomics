---
name: osqp-convex-optimization-solving
description: Use when estimating per-sample reaction fluxes from normalized MRAS scores
  with nutrient-aware bounds, i.e., when an E-Flux-constrained, parsimonious-FBA solution
  is needed over a genome-scale metabolic model.
when_to_use_negative:
- Input is single-cell data with cell-type composition — use compute_sc_flux for community
  fluxes instead of per-sample bulk QP
- You only need reaction activity scores (MRAS) from gene expression — use calculate_reaction_score;
  the QP solver is the downstream FBA step
- No medium definition is available, since uptake must be restricted to medium-defined
  exchange reactions for a nutrient-aware solution
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: osqp
  role: R optimization package that solves the convex quadratic program with the S·v
    ≈ 0 constraints to produce the parsimonious flux distribution
- name: METAFlux
  role: R package that builds the QP (E-Flux bounds, medium-restricted exchange, biomass-plus-parsimony
    objective) and calls osqp via compute_flux
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment; METAFlux is loaded as an R library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:504476d3f46c9170f0b544cbd1391df2a1ed6e5150f3f110e37b25ee38074f00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - osqp
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# osqp-convex-optimization-solving

## Summary

Solve the convex quadratic program behind METAFlux's FBA step: minimize 1/2 v^T v − a C^T v (weight a = 10000) subject to S·v ≈ 0 over the Human-GEM model, using the osqp solver to obtain a parsimonious, biomass-oriented flux distribution.

## When to use

Use when estimating per-sample reaction fluxes from normalized MRAS scores with nutrient-aware bounds, i.e., when an E-Flux-constrained, parsimonious-FBA solution is needed over a genome-scale metabolic model.

## When NOT to use

- Input is single-cell data with cell-type composition — use compute_sc_flux for community fluxes instead of per-sample bulk QP
- You only need reaction activity scores (MRAS) from gene expression — use calculate_reaction_score; the QP solver is the downstream FBA step
- No medium definition is available, since uptake must be restricted to medium-defined exchange reactions for a nutrient-aware solution

## Inputs

- Single-sample normalized MRAS reaction scores (from calculate_reaction_score)
- Human-GEM metabolic model (13,082 reactions, 8,378 metabolites; stoichiometric matrix S)
- Medium file defining exchange-reaction bounds (e.g., human_blood for human-derived samples or cell_medium for cell lines)

## Outputs

- Per-reaction parsimonious flux vector for all 13,082 Human-GEM reactions (writable as fluxes.csv, indexed by Human-GEM Reaction ID)
- Optionally cubic-root-normalized flux scores

## How to apply

First set flux bounds E-Flux style: upper bound = normalized MRAS per reaction, lower bound = −MRAS for reversible reactions and 0 otherwise, further restricted by medium-defined exchange bounds so only medium metabolites (e.g., human_blood, 1648 exchange reactions) can be taken up. Then build the convex QP objective min 1/2 v^T v − a C^T v with weight a = 10000, which simultaneously optimizes the biomass objective (biomass_human from Human1) and minimizes the sum of squared fluxes, constrained by S·v ≈ 0 with an equality tolerance. Solve the QP over all 13,082 Human-GEM reactions using the osqp R package. Optionally apply cubic-root normalization (sign(x)·|x|^(1/3)) to the solved fluxes and write the per-reaction vector, indexed by Human-GEM Reaction ID, to fluxes.csv.

## Related tools

- **osqp** (R optimization package that solves the convex quadratic program with the S·v ≈ 0 constraints to produce the parsimonious flux distribution)
- **METAFlux** (R package that builds the QP (E-Flux bounds, medium-restricted exchange, biomass-plus-parsimony objective) and calls osqp via compute_flux) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment; METAFlux is loaded as an R library)

## Examples

```
flux <- compute_flux(mras = scores, medium = human_blood)
```

## Evaluation signals

- Output contains exactly one flux value for each of the 13,082 Human-GEM reactions
- Exchange reactions show positive flux for metabolite release and negative flux for uptake
- Solution satisfies the stoichiometric constraint S·v ≈ 0 within the configured equality tolerance
- Solved flux vector is reproducible against the reference compute_flux(mras=scores, medium=human_blood) result on the bulk_test_example toy dataset
- Medium-restricted exchange bounds are respected: reactions for metabolites absent from the medium file carry zero uptake flux

## Limitations

- Uptake is restricted to metabolites defined in the chosen medium file, so customizing the medium (adding/deleting nutrient rows) directly changes feasible fluxes
- The objective is the biomass_human generic biomass reaction from Human1 plus parsimony (sum of squared fluxes), which may not capture all metabolic objectives
- No software version number for METAFlux is stated in the provided text; osqp must be installed separately as a dependency

## Evidence

- [readme] osqp dependency: [evidence span withheld — anchor status: external_doc]
- [methods] Human-GEM model scale: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] flux computation scope: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [discussion] objective function: "current objective function of METAFlux maximizes the flux of generic biomass reaction (biomass_human from Human1)"
- [methods] optional normalization: "optional: flux scores cubic root normalization"
