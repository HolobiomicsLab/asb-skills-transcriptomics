---
name: reaction-activity-score-flux-bound-setting
description: Use when converting a gene-by-sample RNA-seq expression matrix into reaction-level
  upper/lower bounds for Human-GEM FBA, i.e., capping each of the 13,082 reaction
  fluxes by its GPR-derived normalized MRAS with a nutrient medium constraint.
when_to_use_negative:
- Input is single-cell RNA-seq needing community/TME fluxes — use compute_sc_flux
  with cell-type fractions and a shared medium instead of per-sample bound setting.
- The goal is downstream pathway-level activity (mean flux of reactions per pathway)
  or metabolite uptake/release lookup via the nutrient lookup files — these consume
  the flux vector rather than set bounds.
- The expression matrix is not gene x sample with human gene symbols as row names;
  GPR-based MRAS computation requires symbol-level inputs.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: METAFlux
  role: Provides calculate_reaction_score (GPR-based MRAS) and compute_flux (FBA with
    MRAS-set bounds and medium exchange restriction) in R
  repo: https://github.com/KChen-lab/METAFlux
- name: osqp
  role: Convex QP solver that solves the parsimonious flux objective (min 1/2 v^T
    v - a C^T v, a=10000) under the MRAS-derived bounds
- name: R
  role: Runtime hosting the METAFlux package, its toy data (bulk_test_example, human_blood,
    cell_medium), and the optional cbrt normalization function
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:eae983c8448d44dc4664dd7649f1747488b49f251470f1434610b6bbb003bb50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - osqp
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# reaction-activity-score-flux-bound-setting

## Summary

Computes Metabolic Reaction Activity Scores (MRAS) from a bulk RNA-seq matrix via GPR rules and sets E-Flux-style flux bounds (upper = normalized MRAS, lower = -MRAS for reversible reactions) on the Human-GEM model before METAFlux's osqp-solved quadratic-programming FBA.

## When to use

Use when converting a gene-by-sample RNA-seq expression matrix into reaction-level upper/lower bounds for Human-GEM FBA, i.e., capping each of the 13,082 reaction fluxes by its GPR-derived normalized MRAS with a nutrient medium constraint.

## When NOT to use

- Input is single-cell RNA-seq needing community/TME fluxes — use compute_sc_flux with cell-type fractions and a shared medium instead of per-sample bound setting.
- The goal is downstream pathway-level activity (mean flux of reactions per pathway) or metabolite uptake/release lookup via the nutrient lookup files — these consume the flux vector rather than set bounds.
- The expression matrix is not gene x sample with human gene symbols as row names; GPR-based MRAS computation requires symbol-level inputs.

## Inputs

- Gene x sample bulk RNA-seq expression matrix with human gene symbols as row names (e.g., bulk_test_example)
- Medium file selecting allowed nutrients: human_blood (human-derived samples) or cell_medium (cell lines)
- Human-GEM genome-scale metabolic model (13,082 reactions, 8,378 metabolites)
- Per-sample normalized MRAS vector from calculate_reaction_score

## Outputs

- Per-reaction flux vector covering all 13,082 Human-GEM reactions (e.g., written to fluxes.csv, indexed by Human-GEM Reaction ID)
- Optionally cubic-root-normalized flux scores, sign(x)*|x|^(1/3)

## How to apply

Compute the single-sample normalized MRAS from the gene x sample matrix (row names must be human gene symbols) with calculate_reaction_score, which aggregates expression onto reactions through Gene-Protein-Reaction rules. Load exactly one medium file — human_blood for human-derived samples or cell_medium for cell lines — alongside Human-GEM (13,082 reactions, 8,378 metabolites). Set bounds E-Flux style: upper bound equals the normalized MRAS per reaction, lower bound equals -MRAS for reversible reactions and 0 otherwise, while medium-defined exchange bounds ensure only medium metabolites can be taken up. Feed these bounds into the convex QP min 1/2 v^T v - a C^T v with a = 10000 under S v ≈ 0, solved by osqp through compute_flux(mras=scores, medium=human_blood), then optionally apply cubic-root normalization sign(x)*|x|^(1/3). Rationale: MRAS scaling ties each reaction's flux capacity to transcriptomic activity, and the medium restriction keeps the parsimonious solution nutrient-aware rather than degenerate.

## Related tools

- **METAFlux** (Provides calculate_reaction_score (GPR-based MRAS) and compute_flux (FBA with MRAS-set bounds and medium exchange restriction) in R) — https://github.com/KChen-lab/METAFlux
- **osqp** (Convex QP solver that solves the parsimonious flux objective (min 1/2 v^T v - a C^T v, a=10000) under the MRAS-derived bounds)
- **R** (Runtime hosting the METAFlux package, its toy data (bulk_test_example, human_blood, cell_medium), and the optional cbrt normalization function)

## Examples

```
scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras = scores, medium = human_blood)
```

## Evaluation signals

- Output flux vector contains exactly one flux value for each of the 13,082 Human-GEM reactions.
- Result is reproducible against compute_flux(mras=scores, medium=human_blood) on the bulk_test_example toy dataset.
- Exchange reactions show positive flux values for release and negative for uptake, and only medium-listed metabolites have nonzero uptake.
- Lower bounds respect reversibility: reversible reactions may carry negative flux (>= -MRAS) while irreversible reactions remain >= 0.
- If applied, cubic-root normalization preserves sign and compresses magnitudes via sign(x)*|x|^(1/3).

## Limitations

- Supports binary experimental conditions only (nutrient presence vs absence), with exactly one medium file chosen per model run.
- No software version numbers for METAFlux, R, or Seurat are stated in the article; reproduction requires pinning the GitHub package.
- Predicted glucose uptake correlated less consistently than other metabolites in the NCI-60 panel, so glucose-flux comparisons warrant caution.
- Flux magnitudes come from a parsimonious QP that minimizes the sum of squared fluxes, so they are not unique mechanistic transport rates.

## Evidence

- [methods] calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction): "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood): "users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood)"
- [methods] optional: flux scores cubic root normalization: "optional: flux scores cubic root normalization"
- [intro] METAFlux can predict 13,082 reaction flux scores for each bulk sample: "METAFlux can predict 13,082 reaction flux scores for each bulk sample"
- [readme] Install the osqp package for optimization using `install.packages('osqp')`: [evidence span withheld — anchor status: external_doc]
