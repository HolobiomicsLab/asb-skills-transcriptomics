---
name: metabolic-reaction-activity-scoring
description: Use when you have a normalized, non-negative gene x sample bulk RNA-seq
  matrix (or bootstrapped single-cell average-expression profiles per cell type) with
  human gene symbols as row names, and need reaction-level activity scores before
  flux estimation.
when_to_use_negative:
- Input is proteomics or metabolomics abundance data (e.g. LC/MS metabolite concentrations)
  rather than RNA-seq gene expression — GPR-based MRAS requires gene-level signal
- Row names of the expression matrix are not human gene symbols (e.g. Ensembl IDs,
  probe IDs, or non-human orthologs), since the GPR rules map HGNC symbols
- You need per-reaction fluxes under a specific nutrient condition — MRAS alone is
  an activity score; fluxes require a subsequent medium-constrained compute_flux/compute_sc_flux
  step
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: METAFlux
  role: Provides calculate_reaction_score (and calculate_avg_exp for single-cell aggregation)
    that computes GPR-based, single-sample normalized MRAS from expression
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Runtime environment for the METAFlux package calls (library(METAFlux))
- name: Seurat
  role: Holds the scRNA-seq object whose cell-type identity is used by calculate_avg_exp
    to build bootstrapped average-expression profiles before scoring
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:25d62afce311a4108db9b60cef863b06dd977a65ca93f47c5e8a4dc51369d413
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# metabolic-reaction-activity-scoring

## Summary

Compute single-sample normalized Metabolic Reaction Activity Scores (MRAS) from a gene x sample expression matrix by aggregating gene expression through Human-GEM Gene-Protein-Reaction (GPR) rules, producing scores for the 13,082 metabolic reactions in Human-GEM. This is the input layer that METAFlux's compute_flux and compute_sc_flux consume to solve flux balance problems.

## When to use

Use when you have a normalized, non-negative gene x sample bulk RNA-seq matrix (or bootstrapped single-cell average-expression profiles per cell type) with human gene symbols as row names, and need reaction-level activity scores before flux estimation.

## When NOT to use

- Input is proteomics or metabolomics abundance data (e.g. LC/MS metabolite concentrations) rather than RNA-seq gene expression — GPR-based MRAS requires gene-level signal
- Row names of the expression matrix are not human gene symbols (e.g. Ensembl IDs, probe IDs, or non-human orthologs), since the GPR rules map HGNC symbols
- You need per-reaction fluxes under a specific nutrient condition — MRAS alone is an activity score; fluxes require a subsequent medium-constrained compute_flux/compute_sc_flux step

## Inputs

- Gene x sample expression matrix with human gene symbols (HGNC) as row names, normalized and non-negative (e.g. bulk_test_example)
- Seurat object for single-cell input, split by patient_id, with a cell-type identity column (e.g. Cell_type)
- Human-GEM GPR mapping implicit in METAFlux covering 13,082 metabolic reactions and 8,378 metabolites

## Outputs

- Single-sample normalized MRAS matrix (13,082 reactions x samples for bulk; 13,082 x cell types per patient for single-cell)
- Score matrix passed as the mras argument to compute_flux or fluxscore argument to compute_sc_flux

## How to apply

Load the gene expression matrix as a gene x sample matrix with human gene symbols as row names (normalized, non-negative); for single-cell data, first build per-cell-type average expression profiles per patient with calculate_avg_exp using stratified bootstrapping (e.g. myident='Cell_type', n_bootstrap per vignette recommendation, seed=1). Call calculate_reaction_score on the matrix, which maps expression through GPR rules onto the 13,082-reaction Human-GEM model (13,082 reactions, 8,378 metabolites) and applies single-sample normalization. The resulting MRAS matrix columns should match the input samples (or cell types within a patient) and rows the 13,082 reactions; pass it directly to compute_flux(mras=scores, medium=...) for bulk samples or as the fluxscore argument to compute_sc_flux for TME communities. Decision point: choose the medium file only downstream (cell_medium for cell lines, human_blood for human-derived samples) — MRAS scoring itself is medium-independent.

## Related tools

- **METAFlux** (Provides calculate_reaction_score (and calculate_avg_exp for single-cell aggregation) that computes GPR-based, single-sample normalized MRAS from expression) — https://github.com/KChen-lab/METAFlux
- **R** (Runtime environment for the METAFlux package calls (library(METAFlux)))
- **Seurat** (Holds the scRNA-seq object whose cell-type identity is used by calculate_avg_exp to build bootstrapped average-expression profiles before scoring)

## Examples

```
library(METAFlux); scores <- calculate_reaction_score(bulk_test_example)
```

## Evaluation signals

- Score matrix has 13,082 rows, one per Human-GEM reaction (13,082 reactions, 8,378 metabolites)
- Columns of the MRAS matrix match the input sample count (bulk) or the number of cell types per patient (single-cell, num_cell entries whose fractions sum to 1)
- Scores are single-sample normalized (comparable across samples/cell types) and non-degenerate, as METAFlux is designed to 'output non-degenerative fluxes'
- Downstream validation: with human_blood medium on 84 TNBC samples, predicted glucose uptake (HMR_9034, negative = uptake) vs FDG-PET SUV should reach Spearman rho = 0.31, P = 0.003; dropping the medium constraint should degrade overall NCI-60 correlation to rho = 0.15
- For single-cell, the downstream flux matrix dimension should equal (num_cell*13082 + 1648) x number_of_bootstrap

## Limitations

- MRAS depends on the Human-GEM objective being the generic biomass reaction (biomass_human from Human1); metabolite-specific performance varies — predicted glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel
- Single-cell scRNA-seq sparsity requires stratified bootstrapped average expression per patient before scoring; no software version numbers for METAFlux, R, or Seurat are stated in the methods
- Installation may require explicitly installing the osqp and dplyr dependencies, and the Seurat pipeline is documented for Seurat V4 only

## Evidence

- [methods] calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction): "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] scores<-calculate_reaction_score(bulk_test_example): "scores<-calculate_reaction_score(bulk_test_example)"
- [methods] The gene expression matrix should be gene x sample matrix where row names are human gene names (gene symbols): "The gene expression matrix should be gene x sample matrix where row names are **human gene names** (gene symbols)"
- [methods] We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [intro] METAFlux can predict 13,082 reaction flux scores for each bulk sample: "METAFlux can predict 13,082 reaction flux scores for each bulk sample"
- [readme] Install the osqp package for optimization using install.packages('osqp'), If you encounter any issue during METAFlux installation.: [evidence span withheld — anchor status: external_doc]
