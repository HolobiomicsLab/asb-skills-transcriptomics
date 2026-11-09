---
name: gpr-based-reaction-activity-scoring
description: Use when you have a bulk gene x sample RNA-seq expression matrix (human
  gene symbols as row names, non-negative normalized values) and need reaction-level
  activity scores to feed compute_flux in a METAFlux/Human-GEM workflow.
when_to_use_negative:
- Input is single-cell data — use calculate_avg_exp plus compute_sc_flux instead of
  bulk MRAS scoring
- You already have MRAS/reaction scores and only need fluxes — proceed directly to
  compute_flux with a medium file
- Expression matrix uses non-human identifiers or contains negative values — it must
  first be converted to human gene symbols and normalized
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: METAFlux
  role: R package providing calculate_reaction_score for GPR-based single-sample normalized
    MRAS computation from bulk expression
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Runtime environment for METAFlux (library(METAFlux)) and the MRAS calculation
    script
- name: Human-GEM
  role: Underlying genome-scale metabolic model whose reaction IDs and GPR rules define
    the reaction set scored
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:a43b18496101fc6702159908fdb6bfbe25bfe75b50b5301f18a2240e3a779bdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - Human-GEM
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gpr-based-reaction-activity-scoring

## Summary

Computes single-sample normalized Metabolic Reaction Activity Scores (MRAS) from a bulk gene expression matrix by resolving Human-GEM Gene-protein-reaction (GPR) rules, producing the 13082-reaction input required by METAFlux's downstream flux balance analysis.

## When to use

Use when you have a bulk gene x sample RNA-seq expression matrix (human gene symbols as row names, non-negative normalized values) and need reaction-level activity scores to feed compute_flux in a METAFlux/Human-GEM workflow.

## When NOT to use

- Input is single-cell data — use calculate_avg_exp plus compute_sc_flux instead of bulk MRAS scoring
- You already have MRAS/reaction scores and only need fluxes — proceed directly to compute_flux with a medium file
- Expression matrix uses non-human identifiers or contains negative values — it must first be converted to human gene symbols and normalized

## Inputs

- Bulk gene expression matrix (gene x sample; row names are human gene symbols; normalized, non-negative values), e.g. data('bulk_test_example')
- Human-GEM model information (13082 metabolic reactions with Reaction IDs, GPR gene associations, reversibility, compartments, SUBSYSTEM pathway labels)

## Outputs

- MRAS matrix of 13082 reactions x samples, keyed by Human-GEM Reaction IDs, one column per sample, no missing or negative scores (e.g. mras_scores.csv)

## How to apply

Load the gene expression matrix (e.g. bulk_test_example) as a gene x sample matrix with human gene symbols as row names and no negative entries, and load Human-GEM (13082 metabolic reactions, 8378 metabolites) with reaction IDs, GPR associations, reversibility, compartments, and SUBSYSTEM labels. Map expression to reactions via GPR rules: for AND logic (multi-subunit complexes) take the minimum expression across subunits, for OR logic (isoenzymes) sum isoenzyme contributions, and apply the promiscuity adjustment by dividing each gene's expression by the number of reactions that gene participates in. Then perform single-sample normalization of each sample's reaction scores to produce the MRAS matrix (13082 reactions x samples). In METAFlux this is wrapped in calculate_reaction_score, whose output is passed directly to compute_flux with a chosen medium (cell_medium or human_blood).

## Related tools

- **METAFlux** (R package providing calculate_reaction_score for GPR-based single-sample normalized MRAS computation from bulk expression) — https://github.com/KChen-lab/METAFlux
- **R** (Runtime environment for METAFlux (library(METAFlux)) and the MRAS calculation script)
- **Human-GEM** (Underlying genome-scale metabolic model whose reaction IDs and GPR rules define the reaction set scored)

## Examples

```
library(METAFlux); data("bulk_test_example"); scores <- calculate_reaction_score(bulk_test_example)
```

## Evaluation signals

- Output mras_scores.csv has exactly 13082 reaction rows keyed by Human-GEM Reaction IDs
- One column per input sample; no missing values and no negative scores in the MRAS matrix
- Row names of the input expression matrix are human gene symbols and all values are normalized and non-negative before scoring
- Reaction scores for AND-logic reactions equal the minimum expression across subunits, and OR-logic reactions aggregate isoenzyme contributions as sums
- MRAS output is single-sample normalized per sample and passes directly as the 'mras' argument to compute_flux without reformatting

## Limitations

- Requires the exact Human-GEM reaction set (13082 reactions); reactions must be keyed by Human-GEM Reaction IDs for downstream FBA
- Gene expression matrix must be normalized with no negative entries — negative or unnormalized values invalidate the scoring
- Promiscuity adjustment divides each gene's expression by the number of reactions it participates in, which redistributes signal across multi-gene reactions
- The MRAS step is bulk-sample specific; single-cell datasets follow a different averaging/bootstrapping route (calculate_avg_exp)
- No software version numbers for METAFlux, R, or Seurat are stated in the provided text

## Evidence

- [methods] GPR mapping via calculate_reaction_score: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] calculate_reaction_score invocation: "scores<-calculate_reaction_score(bulk_test_example)"
- [methods] Human-GEM reaction count: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] Input matrix requirements: "The gene expression matrix should be gene x sample matrix where row names are **human gene names** (gene symbols)"
- [methods] Downstream flux step: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [intro] Output dimensionality: "METAFlux can predict 13,082 reaction flux scores for each bulk sample"
