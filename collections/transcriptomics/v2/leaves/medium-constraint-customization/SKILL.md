---
name: medium-constraint-customization
description: Use when running METAFlux FBA on bulk RNA-seq MRAS scores and you must
  choose between cell_medium, human_blood, a custom-edited medium file, or a no-medium
  free-exchange control (known to yield only ρ=0.15 on NCI-60).
when_to_use_negative:
- Input is already solved flux data or metabolite abundance measurements rather than
  gene-expression-derived MRAS scores
- Single-cell community fluxes, where compute_sc_flux handles medium differently via
  cell-type fractions
- When the goal is medium-agnostic uptake/excretion inference — note the paper's control
  shows this condition alone yields poor correlation with measured fluxes (ρ=0.15
  on NCI-60)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: METAFlux
  role: R package providing compute_flux, cell_medium and human_blood medium files,
    and the nutrient lookup file with 1,648 exchange reactions
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment for METAFlux (library(METAFlux), data(cell_medium),
    compute_flux calls)
- name: Human-GEM
  role: Underlying genome-scale metabolic model (13,082 reactions, 8,378 metabolites)
    whose reactions the medium constrains
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:869e8db4af2e75ffbf962f6feca3933ffdb310a259be7e90d986f66009fa3e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - Human-GEM
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# medium-constraint-customization

## Summary

Configure the nutrient exchange medium (or deliberately remove it) in METAFlux's compute_flux step to control which of the 1,648 nutrient exchange reactions are open for uptake/excretion, since medium constraints directly determine predicted flux accuracy.

## When to use

Use when running METAFlux FBA on bulk RNA-seq MRAS scores and you must choose between cell_medium, human_blood, a custom-edited medium file, or a no-medium free-exchange control (known to yield only ρ=0.15 on NCI-60).

## When NOT to use

- Input is already solved flux data or metabolite abundance measurements rather than gene-expression-derived MRAS scores
- Single-cell community fluxes, where compute_sc_flux handles medium differently via cell-type fractions
- When the goal is medium-agnostic uptake/excretion inference — note the paper's control shows this condition alone yields poor correlation with measured fluxes (ρ=0.15 on NCI-60)

## Inputs

- MRAS reaction activity score matrix from calculate_reaction_score (13,082 reactions x samples)
- Medium file: cell_medium (for cell line models) or human_blood (for human-derived samples), containing nutrient exchange rows from the 1,648-reaction nutrient lookup file

## Outputs

- Predicted relative flux scores for all 13,082 Human-GEM reactions per sample
- Optionally cubic-root-normalized flux scores

## How to apply

After computing single-sample normalized MRAS with calculate_reaction_score, load exactly one medium file before calling compute_flux: data(cell_medium) for cell line models or data(human_blood) for human-derived samples. Customize the medium by deleting or adding nutrient rows — to remove a nutrient, delete the entire row where that nutrient is located. For a no-medium control, bypass the medium argument so all 1,648 nutrient exchange reactions are freely open; the paper's benchmark shows this drops overall Spearman correlation on NCI-60 matched flux data to ρ=0.15, so medium constraints are essential for accuracy. Run compute_flux(mras=scores, medium=...) to solve the flux balance problem over the 13,082 Human-GEM reactions, then optionally apply cubic-root normalization (sign(x)*abs(x)^(1/3)).

## Related tools

- **METAFlux** (R package providing compute_flux, cell_medium and human_blood medium files, and the nutrient lookup file with 1,648 exchange reactions) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment for METAFlux (library(METAFlux), data(cell_medium), compute_flux calls))
- **Human-GEM** (Underlying genome-scale metabolic model (13,082 reactions, 8,378 metabolites) whose reactions the medium constrains)

## Examples

```
data(cell_medium); flux <- compute_flux(mras=scores, medium = cell_medium)
```

## Evaluation signals

- Overall Spearman correlation between predicted fluxes and matched NCI-60 measured fluxes is substantially higher than the no-medium baseline of ρ=0.15
- Exactly one medium file was used per model run (cell_medium OR human_blood), not both
- Predicted flux vector covers all 13,082 reactions of the Human-GEM model
- Fluxes are non-degenerative and nutrient-aware, i.e., constrained to the exchange reactions present in the chosen medium file
- Custom medium edits preserve the file's row structure so compute_flux can parse the exchange reaction IDs

## Limitations

- Predicted glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel
- Omitting the medium constraint (free uptake/excretion of all 1,648 exchange reactions) reduces overall Spearman correlation on NCI-60 to ρ=0.15 and significantly lowers per-metabolite and per-cell-line correlations
- No version numbers are stated for METAFlux, R, or Seurat; installing osqp and dplyr may be needed for optimization

## Evidence

- [methods] users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood): "users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood)"
- [methods] To delete nutrients, simply delete the entire row where that nutrient is located: "To delete nutrients, simply delete the entire row where that nutrient is located"
- [methods] ‘Nutrient look up files’ contains 1648 exchange reactions: "‘Nutrient look up files’ contains 1648 exchange reactions"
- [discussion] the predicted glucose uptake does not correlate as consistently as other metabolites in NCI-60 panel: "the predicted glucose uptake does not correlate as consistently as other metabolites in NCI-60 panel"
- [methods] We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
