---
name: medium-exchange-reaction-configuration
description: Use when running METAFlux on bulk or single-cell RNA-seq data and the
  model must be constrained to a defined nutrient environment (cell culture vs. human-derived
  samples), or when customizing which nutrients are available for uptake.
when_to_use_negative:
- The analysis goal is unconstrained FBA where all exchange reactions are open — the
  medium file deliberately restricts uptake.
- Only one model should be run per analysis; choosing both cell_medium and human_blood
  is explicitly not supported — pick the one matching the sample type.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: METAFlux
  role: R package whose compute_flux/compute_sc_flux functions consume the medium-defined
    exchange bounds
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Environment for loading the medium data files and editing nutrient rows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:d128b7669874455e5ff8bc33c574e5a4169b75364a793b67529b13e6401e2d31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# medium-exchange-reaction-configuration

## Summary

Configure the nutrient medium file (e.g. human_blood or cell_medium) so that METAFlux's FBA only permits uptake of medium metabolites through their exchange reactions, making flux predictions nutrient-aware.

## When to use

Use when running METAFlux on bulk or single-cell RNA-seq data and the model must be constrained to a defined nutrient environment (cell culture vs. human-derived samples), or when customizing which nutrients are available for uptake.

## When NOT to use

- The analysis goal is unconstrained FBA where all exchange reactions are open — the medium file deliberately restricts uptake.
- Only one model should be run per analysis; choosing both cell_medium and human_blood is explicitly not supported — pick the one matching the sample type.

## Inputs

- Medium file (cell_medium for cell line model or human_blood for human-derived samples)
- Nutrient lookup files containing 1648 exchange reactions (Human-GEM Reaction IDs)

## Outputs

- Exchange-reaction bounds restricting uptake to medium metabolites, consumed by compute_flux or compute_sc_flux

## How to apply

Choose exactly one medium file: cell_medium for cell line models or human_blood for human-derived samples. The medium defines bounds on exchange reactions in the Human-GEM model (13,082 reactions, 8,378 metabolites) so that only medium metabolites can be taken up during the convex QP solve. The nutrient lookup files list 1,648 exchange reactions and can be searched to obtain the metabolite exchange Reaction ID when targeting a specific uptake or release. To customize the medium, delete the entire row where an unwanted nutrient is located, or add nutrient rows to grant uptake of additional metabolites. This medium constraint is applied together with E-Flux style MRAS-derived bounds before solving the QP with osqp.

## Related tools

- **METAFlux** (R package whose compute_flux/compute_sc_flux functions consume the medium-defined exchange bounds) — https://github.com/KChen-lab/METAFlux
- **R** (Environment for loading the medium data files and editing nutrient rows)

## Examples

```
data(human_blood); flux <- compute_flux(mras = scores, medium = human_blood)
```

## Evaluation signals

- Only one medium file (cell_medium or human_blood) was loaded for the run
- Exchange reactions for metabolites absent from the medium show no uptake (non-negative/zero flux on the uptake side) in the solved flux vector
- Uptake/release directionality of medium exchange reactions matches expectations (positive for release, negative for uptake)
- Custom medium edits removed entire rows for deleted nutrients rather than partial edits

## Limitations

- The nutrient lookup files cover 1,648 exchange reactions, so not all Human-GEM exchanges may be medium-configurable
- No version numbers are stated for METAFlux, R, or Seurat in the provided text, so medium file formats may vary across package versions

## Evidence

- [methods] Medium file selection: "users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood)"
- [methods] Nutrient lookup files: "‘Nutrient look up files’ contains 1648 exchange reactions"
- [methods] Medium customization: "To delete nutrients, simply delete the entire row where that nutrient is located"
- [methods] Exchange reaction lookup: "If one is strictly interested in metabolite uptake or release, one can search “nutrient lookup file” to obtain the metabolite exchange Reaction ID"
- [discussion] Medium-restricted uptake: "enables a broad spectrum of metabolic flux characterization from bulk, flow-sorted and single-cell RNA expression data, in a medium and TME"
- [readme] Installation context: [evidence span withheld — anchor status: external_doc]
