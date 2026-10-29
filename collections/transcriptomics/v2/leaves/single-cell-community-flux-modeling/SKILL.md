---
name: single-cell-community-flux-modeling
description: Use when you have a patient-partitioned scRNA-seq object with cell-type
  annotations and want community-aware metabolic fluxes per cell type (e.g. glucose
  uptake across myeloid, B, T, epithelial cells) rather than per-cell or bulk fluxes.
when_to_use_negative:
- Input is bulk RNA-seq rather than annotated single-cell data — use METAFlux::compute_flux
  instead of compute_sc_flux
- Data lacks cell-type annotations or patient/TME grouping needed to define the community
  and its fractions
- Per-single-cell flux estimates are required — METAFlux calculates fluxes at the
  cell cluster/type level, not per individual cell
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: METAFlux
  role: Core R package providing calculate_avg_exp, calculate_reaction_score, and
    compute_sc_flux for community flux balance analysis against Human-GEM
  repo: https://github.com/KChen-lab/METAFlux
- name: Seurat
  role: Holds the annotated single-cell object split by patient_id; interface used
    to build per-cell-type average expression profiles (Seurat V4)
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment for the METAFlux workflow (library(METAFlux))
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:046984fba14f197fdc82a72994fd6de46f0c46500e4d86bce85b53415b8f3dce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-community-flux-modeling

## Summary

Model a tumor microenvironment as a multi-cell-type metabolic community with METAFlux: aggregate scRNA-seq expression per cell type per patient, compute MRAS against the 13,082-reaction Human-GEM, and run FBA over the whole TME to obtain per-cell-type reaction fluxes (e.g. glucose uptake via HMR_9034).

## When to use

Use when you have a patient-partitioned scRNA-seq object with cell-type annotations and want community-aware metabolic fluxes per cell type (e.g. glucose uptake across myeloid, B, T, epithelial cells) rather than per-cell or bulk fluxes.

## When NOT to use

- Input is bulk RNA-seq rather than annotated single-cell data — use METAFlux::compute_flux instead of compute_sc_flux
- Data lacks cell-type annotations or patient/TME grouping needed to define the community and its fractions
- Per-single-cell flux estimates are required — METAFlux calculates fluxes at the cell cluster/type level, not per individual cell

## Inputs

- Seurat scRNA-seq object with patient_id and Cell_type metadata (e.g. 25,536-cell metastatic LUAD dataset)
- human_blood medium file (64 blood metabolites) from the METAFlux package
- Gene x sample expression matrix with human gene symbols as row names
- Nutrient lookup file mapping metabolites to 1,648 exchange reaction IDs (e.g. HMR_9034 for glucose)

## Outputs

- Flux matrix with dimension (num_cell*13082 + 1648) x number_of_bootstrap reaction fluxes per cell type
- Per-cell-type Metabolic Reaction Activity Scores (MRAS)
- Cell-type-specific glucose uptake fluxes (HMR_9034) for Cuzick trend testing

## How to apply

Split the Seurat object by patient_id so fluxes are modeled within each tumor microenvironment, then call METAFlux::calculate_avg_exp with myident='Cell_type' and stratified bootstrapping (n_bootstrap 50-100 per the vignette, seed=1) to build per-cell-type average expression profiles per patient. Convert expression to single-sample-normalized Metabolic Reaction Activity Scores with METAFlux::calculate_reaction_score using Gene-Protein-Reaction rules against the Human-GEM (13,082 reactions, 8,378 metabolites). Model the whole TME as one community with METAFlux::compute_sc_flux, passing num_cell equal to the number of cell types, cell-type fractions that sum to 1 and are ordered to match the score matrix columns, fluxscore=scores, and the human_blood medium (64 blood metabolites) for human-derived samples. Extract the internal_medium glucose exchange reaction HMR_9034 (via the 1648-reaction nutrient lookup file) and apply cubic-root normalization sign(x)*|x|^(1/3) before statistical comparison across cell types.

## Related tools

- **METAFlux** (Core R package providing calculate_avg_exp, calculate_reaction_score, and compute_sc_flux for community flux balance analysis against Human-GEM) — https://github.com/KChen-lab/METAFlux
- **Seurat** (Holds the annotated single-cell object split by patient_id; interface used to build per-cell-type average expression profiles (Seurat V4)) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment for the METAFlux workflow (library(METAFlux)))

## Examples

```
mean_exp=calculate_avg_exp(myseurat=sc_test_example,myident='Cell_type',n_bootstrap=3,seed=1); flux=compute_sc_flux(num_cell=4,fraction=c(0.1,0.3,0.3,0.3),fluxscore=scores,medium=human_blood)
```

## Evaluation signals

- Flux matrix dimension equals (num_cell*13082 + 1648) x number_of_bootstrap
- Cuzick nonparametric trend test across Epithelial < T cell < B cell < Myeloid yields P < 0.0001 with monotonic ordering Myeloid > B cell > T cell > Epithelial (metastatic LUAD glucose uptake)
- Cell-type fractions passed to compute_sc_flux sum to 1 and follow the column order of the MRAS score matrix
- HMR_9034 glucose exchange is extracted from the internal_medium section and normalized with cbrt <- function(x) sign(x) * abs(x)^(1/3)
- Exactly one medium file is used (human_blood for human-derived samples; cell_medium only for cell lines)

## Limitations

- Fluxes are predicted at cell cluster/type level, not for individual cells; no METAFlux version number is stated in the text or README
- Predicted glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel
- Only one medium file may be used per model; cell fractions must be supplied externally and must sum to 1
- Installation requires osqp and dplyr; Seurat interface is validated for Seurat V4 only

## Evidence

- [methods] compute_sc_flux with cell type fractions: "flux=compute_sc_flux(num_cell = 4,fraction =c(0.1,0.3,0.3,0.3),fluxscore=scores,medium = human_blood)"
- [methods] Human-GEM model size: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] stratified bootstrap averaging: "Create an average expression profile for stratified bootstrapped samples for this patient"
- [intro] output dimensionality: "for each single cell data, it can predict (13,082 × number of cell-type/cluster + 1648) reaction flux scores"
- [readme] community capabilities: [evidence span withheld — anchor status: external_doc]
