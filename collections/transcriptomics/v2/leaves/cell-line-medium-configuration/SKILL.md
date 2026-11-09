---
name: cell-line-medium-configuration
description: Use when predicting metabolic fluxes from cancer cell-line gene expression
  (e.g., NCI-60) with METAFlux and a cell-culture medium constraint must be chosen
  or customized (nutrient presence vs absence).
when_to_use_negative:
- Input is scRNA-seq from human-derived (e.g., tumor microenvironment or blood) samples
  where human_blood, not cell_medium, is the appropriate medium
- The goal is a nutrient-unconstrained or condition-free flux estimate; METAFlux is
  customized for binary nutrient presence/absence conditions
- Modeling an organism or model scope other than the Human-GEM model (13082 metabolic
  reactions, 8378 metabolites)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: METAFlux
  role: Provides the cell_medium data object, compute_flux() and compute_sc_flux()
    functions, and the Human-GEM-based flux balance framework the medium constrains
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Runtime environment in which METAFlux is loaded (library(METAFlux)) and the
    medium is customized and passed to compute_flux()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:cbc221af57b37f7fe3b9d4eef931ace3e379279625ef60918ef256ca863102cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cell-line-medium-configuration

## Summary

Configure the nutrient environment for METAFlux flux balance analysis on cell-line RNA-seq data by selecting the cell_medium medium file (44 metabolites) and optionally customizing it by adding or deleting nutrient rows before running compute_flux against the Human-GEM model (13082 reactions, 8378 metabolites).

## When to use

Use when predicting metabolic fluxes from cancer cell-line gene expression (e.g., NCI-60) with METAFlux and a cell-culture medium constraint must be chosen or customized (nutrient presence vs absence).

## When NOT to use

- Input is scRNA-seq from human-derived (e.g., tumor microenvironment or blood) samples where human_blood, not cell_medium, is the appropriate medium
- The goal is a nutrient-unconstrained or condition-free flux estimate; METAFlux is customized for binary nutrient presence/absence conditions
- Modeling an organism or model scope other than the Human-GEM model (13082 metabolic reactions, 8378 metabolites)

## Inputs

- R data object cell_medium (44 metabolites) packaged with the METAFlux R package
- Gene x sample expression matrix with human gene symbols as row names (non-negative, normalized)
- MRAS reaction activity scores from calculate_reaction_score()
- Nutrient lookup file (1648 exchange reactions) for identifying metabolite exchange Reaction IDs

## Outputs

- Medium-constrained flux scores for 13082 Human-GEM reactions per cell-line sample from compute_flux()
- Customized medium table (rows added/deleted) if the default cell_medium was modified

## How to apply

Load the packaged medium with data(cell_medium) in R after library(METAFlux); this is the medium designated for the cell line model, whereas human_blood is reserved for human-derived samples. METAFlux fits binary experimental conditions (nutrient presence vs absence), so the medium determines which exchange reactions can carry flux. To customize, delete the entire row where an unwanted nutrient is located, or add nutrient rows for supplemented metabolites; the 'Nutrient look up files' contain 1648 exchange reactions for identifying metabolite exchange Reaction IDs. Pass the chosen medium to compute_flux(mras=scores, medium=cell_medium) so the flux balance problem over the 13082 Human-GEM reactions is constrained by that nutrient set. Use only one medium file per model run.

## Related tools

- **METAFlux** (Provides the cell_medium data object, compute_flux() and compute_sc_flux() functions, and the Human-GEM-based flux balance framework the medium constrains) — https://github.com/KChen-lab/METAFlux
- **R** (Runtime environment in which METAFlux is loaded (library(METAFlux)) and the medium is customized and passed to compute_flux())

## Examples

```
library(METAFlux); data("cell_medium"); flux <- compute_flux(mras = scores, medium = cell_medium)
```

## Evaluation signals

- The medium object loaded is cell_medium (44 metabolites) when modeling cell lines, not human_blood
- compute_flux(mras=scores, medium=cell_medium) runs without unbounded infeasibility and returns fluxes for all 13082 reactions
- Predicted exchange fluxes can be extracted via nutrient lookup Reaction IDs (e.g., glucose uptake via HMR_9034) consistent with the configured medium
- In the NCI-60 benchmark, downstream Spearman correlations of predicted vs experimental fluxes exceed the ecGEMs baseline (0.74 vs 0.45)

## Limitations

- Only one medium file should be chosen per model (cell_medium OR human_blood); mixing or double-applying media is not supported by the workflow
- METAFlux is customized for binary experimental conditions (nutrient presence vs absence); continuous nutrient concentration gradients are not represented
- Predicted glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel
- No software version number for METAFlux is stated in the provided text, so reproducibility of medium-package contents may vary across installs

## Evidence

- [methods] cell_medium: "data("cell_medium")# for cell line model"
- [methods] one medium per model: "users will only need to run one model by choosing only **one** medium file(either cell medium or human_blood)"
- [methods] customizing nutrients: "To delete nutrients, simply delete the entire row where that nutrient is located"
- [methods] nutrient lookup: "‘Nutrient look up files’ contains 1648 exchange reactions"
- [intro] binary conditions: "METAFlux only requires gene expression data as input and is customized to fit binary experimental conditions (nutrient presence vs absence)."
