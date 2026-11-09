---
name: mras-reaction-activity-scoring
description: Use when you have a normalized, non-negative bulk gene x sample expression
  matrix with human gene symbols as row names and need reaction-level activity scores
  as the input to metabolic flux prediction (FBA) rather than gene-level expression.
when_to_use_negative:
- Input is already a reaction-level flux matrix or pathway activity table (scoring
  would be redundant)
- Expression matrix uses probe IDs, Ensembl IDs, or non-human gene identifiers without
  first mapping to human gene symbols
- Working with single-cell data where you need cluster-averaged profiles via calculate_avg_exp
  and compute_sc_flux instead of bulk per-sample MRAS
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: METAFlux
  role: R package providing calculate_reaction_score, which applies GPR rules to convert
    gene expression into single-sample normalized MRAS
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment; METAFlux is installed via devtools::install_github('KChen-lab/METAFlux')
    with osqp and dplyr dependencies
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:2ac3bb1cf54237c622c2fb698d20b99d92d00e3353f87eb0f0fe5dddd33bc1fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mras-reaction-activity-scoring

## Summary

Computes single-sample normalized Metabolic Reaction Activity Scores (MRAS) from a gene x sample bulk RNA-seq matrix by collapsing gene expression onto 13,082 Human-GEM reactions via Gene-Protein-Reaction (GPR) rules, producing the MRAS input required by METAFlux's compute_flux step.

## When to use

Use when you have a normalized, non-negative bulk gene x sample expression matrix with human gene symbols as row names and need reaction-level activity scores as the input to metabolic flux prediction (FBA) rather than gene-level expression.

## When NOT to use

- Input is already a reaction-level flux matrix or pathway activity table (scoring would be redundant)
- Expression matrix uses probe IDs, Ensembl IDs, or non-human gene identifiers without first mapping to human gene symbols
- Working with single-cell data where you need cluster-averaged profiles via calculate_avg_exp and compute_sc_flux instead of bulk per-sample MRAS

## Inputs

- Normalized, non-negative gene x sample bulk RNA-seq expression matrix with human gene symbols (HGNC) as row names (e.g., bulk_test_example format)
- Human-GEM Gene-Protein-Reaction (GPR) rules bundled in the METAFlux R package

## Outputs

- Reactions x samples matrix of single-sample normalized Metabolic Reaction Activity Scores (MRAS) covering the 13,082 Human-GEM reactions

## How to apply

Load the expression matrix as a gene x sample matrix with human gene symbols as row names; non-negative normalized values are required since MRAS is derived directly from expression magnitudes. Call METAFlux::calculate_reaction_score on this matrix, which uses the Human-GEM GPR rules to aggregate expression of genes encoding subunits of each of the 13,082 metabolic reactions into a single-sample normalized Metabolic Reaction Activity Score per sample. Verify the output is a reactions x samples score matrix matching the Human-GEM reaction space (13,082 reactions) before passing it to compute_flux with a medium file (human_blood for human-derived samples, cell_medium for cell lines). The rationale is that gene expression alone is not flux: GPR-based scoring maps expression onto reaction activity so that flux balance analysis can operate on reaction-level quantities.

## Related tools

- **METAFlux** (R package providing calculate_reaction_score, which applies GPR rules to convert gene expression into single-sample normalized MRAS) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment; METAFlux is installed via devtools::install_github('KChen-lab/METAFlux') with osqp and dplyr dependencies)

## Examples

```
library(METAFlux); scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras=scores, medium=human_blood)
```

## Evaluation signals

- Output matrix dimensions equal 13,082 reactions (Human-GEM) x number of input samples
- All MRAS values are finite and the matrix is compatible as the `mras` argument of METAFlux::compute_flux
- Input row names are human gene symbols matching the GPR rule identifiers; no NA or zero-row output
- Single-sample normalization means each sample's scores are computed independently — no cross-sample batch effect introduced at this stage

## Limitations

- METAFlux requires non-negative, normalized expression; degenerate or negative-corrected values upstream will corrupt MRAS
- No software version number for METAFlux is stated in the provided text; pin the GitHub commit for reproducibility
- MRAS reflects reaction activity proxy from expression only — it does not capture post-translational or allosteric metabolic regulation

## Evidence

- [methods] methods: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [intro] intro: "METAFlux can predict 13,082 reaction flux scores for each bulk sample"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
