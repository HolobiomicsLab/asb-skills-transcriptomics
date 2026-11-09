---
name: gene-expression-matrix-preprocessing
description: Use when the input is raw or non-conforming bulk RNA-seq expression data
  that must be reshaped into a gene (symbol) x sample matrix before reaction activity
  scoring or flux analysis.
when_to_use_negative:
- Input is already a validated gene x sample MRAS matrix or reaction activity score
  table keyed by Human-GEM Reaction IDs — proceed directly to compute_flux
- The analysis goal is flux estimation from metabolomics measurements (LC/MS, Seahorse
  XF) rather than transcriptome-derived flux inference
- Data are proteomics or metabolite abundance tables without gene-symbol row names,
  which cannot be mapped through GPR rules
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: METAFlux
  role: R package providing calculate_reaction_score and calculate_avg_exp applied
    to the preprocessed expression matrix; installed via devtools::install_github('KChen-lab/METAFlux')
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment; the matrix checks and GPR mapping are implemented in
    R (library(METAFlux))
- name: Seurat
  role: Used to hold and interface single-cell expression data (Seurat V4) for per-patient
    averaging/bootstrapping before scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:1f8df05090f43bbca0a7f0c92e1f1e859e4e63042eda4c74ca44b4ea5ee80405
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-expression-matrix-preprocessing

## Summary

Prepare a bulk RNA-seq gene expression matrix into the gene x sample format METAFlux requires — human gene symbols as row names, normalized values with no negative entries — so that downstream GPR-based MRAS calculation over the Human-GEM's 13082 reactions is valid.

## When to use

Use when the input is raw or non-conforming bulk RNA-seq expression data that must be reshaped into a gene (symbol) x sample matrix before reaction activity scoring or flux analysis.

## When NOT to use

- Input is already a validated gene x sample MRAS matrix or reaction activity score table keyed by Human-GEM Reaction IDs — proceed directly to compute_flux
- The analysis goal is flux estimation from metabolomics measurements (LC/MS, Seahorse XF) rather than transcriptome-derived flux inference
- Data are proteomics or metabolite abundance tables without gene-symbol row names, which cannot be mapped through GPR rules

## Inputs

- bulk RNA-seq gene expression matrix (gene x sample) with human gene symbols as row names, e.g. bulk_test_example
- single-cell Seurat object (e.g. sc_test_example) with cell-type annotations for averaging/bootstrapping
- Human-GEM model gene associations (13082 reactions) used to validate that gene symbols are mappable

## Outputs

- Normalized, non-negative gene x sample expression matrix keyed by human gene symbols, ready for calculate_reaction_score
- For scRNA-seq: averaged expression profiles per cell type from stratified bootstrapped samples (calculate_avg_exp output)

## How to apply

Load the expression data and verify it is a gene x sample matrix whose row names are human gene symbols (HGNC symbols), since METAFlux maps these symbols onto Human-GEM GPR associations covering 13082 metabolic reactions. Ensure values are normalized and contain no negative entries, because reaction activity scoring (AND logic takes the minimum across subunits, OR logic sums isoenzyme contributions, with a promiscuity adjustment dividing each gene's expression by the number of reactions it participates in) assumes non-negative, comparable expression scales. For single-cell workflows, first collapse per-cell profiles: create an average expression profile for stratified bootstrapped samples per patient (calculate_avg_exp with an identity column such as Cell_type and a fixed seed). The cleaned matrix is then fed directly to calculate_reaction_score to produce the single-sample-normalized MRAS matrix.

## Related tools

- **METAFlux** (R package providing calculate_reaction_score and calculate_avg_exp applied to the preprocessed expression matrix; installed via devtools::install_github('KChen-lab/METAFlux')) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment; the matrix checks and GPR mapping are implemented in R (library(METAFlux)))
- **Seurat** (Used to hold and interface single-cell expression data (Seurat V4) for per-patient averaging/bootstrapping before scoring)

## Examples

```
library(METAFlux); data("bulk_test_example"); scores <- calculate_reaction_score(bulk_test_example)
```

## Evaluation signals

- Row names of the matrix are human gene symbols (not Ensembl IDs or probe IDs) so they resolve against Human-GEM GPR gene associations
- Matrix orientation is gene x sample (genes as rows, one column per sample), not the transpose
- All expression values are normalized and non-negative — no missing or negative entries anywhere in the matrix
- Downstream calculate_reaction_score output (mras_scores.csv) contains 13082 reaction rows keyed by Human-GEM Reaction IDs with one column per sample and no missing or negative scores
- For scRNA-seq, bootstrapped averages use a fixed seed (e.g. seed=1, n_bootstrap=3) so repeated runs are reproducible

## Limitations

- No software version numbers for METAFlux, R, or Seurat are stated; Seurat V4 is specified for the single-cell pipeline
- The promiscuity adjustment depends on knowing how many reactions each gene participates in, so unmapped gene symbols silently drop out of GPR coverage
- Single-cell processing works at cell-cluster level via averaged profiles, not per-cell fluxes, and bootstrapped averages introduce seed-dependent variability

## Evidence

- [methods] gene x sample matrix with human gene symbols as row names: "The gene expression matrix should be gene x sample matrix where row names are **human gene names** (gene symbols)"
- [methods] MRAS calculated from gene expression via GPR: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] bootstrapped average expression per patient: "Create an average expression profile for stratified bootstrapped samples for this patient"
- [methods] Human-GEM model scale: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [readme] installation via devtools from GitHub: [evidence span withheld — anchor status: external_doc]
