---
name: single-sample-transcriptomic-normalization
description: Use when converting a bulk RNA-seq gene x sample matrix (human gene symbols
  as row names, non-negative normalized values) into sample-normalized reaction activity
  scores (MRAS) that feed METAFlux's compute_flux step.
when_to_use_negative:
- Input is already a flux or reaction score matrix — apply compute_flux directly instead
  of recomputing MRAS
- Input is single-cell data — use the calculate_avg_exp / compute_sc_flux community
  workflow rather than bulk single-sample MRAS normalization
- The expression matrix contains raw counts or negative entries (e.g. from batch correction)
  — normalize/transform first, since MRAS expects non-negative normalized values
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: METAFlux
  role: R package whose calculate_reaction_score function computes the single-sample
    normalized MRAS from gene expression using GPR rules
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Language environment in which the METAFlux library and GPR mapping/normalization
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:d740da6e86ed106de8bb959aa90bbb63551c2a064c1768be94fb0796635f141a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-sample-transcriptomic-normalization

## Summary

Per-sample normalization of gene-expression-derived metabolic reaction activity scores (MRAS) in METAFlux, where each Human-GEM reaction's score is computed from GPR-mapped gene expression and then normalized within each sample to yield the reaction-level input (13082 reactions x samples) for flux balance analysis.

## When to use

Use when converting a bulk RNA-seq gene x sample matrix (human gene symbols as row names, non-negative normalized values) into sample-normalized reaction activity scores (MRAS) that feed METAFlux's compute_flux step.

## When NOT to use

- Input is already a flux or reaction score matrix — apply compute_flux directly instead of recomputing MRAS
- Input is single-cell data — use the calculate_avg_exp / compute_sc_flux community workflow rather than bulk single-sample MRAS normalization
- The expression matrix contains raw counts or negative entries (e.g. from batch correction) — normalize/transform first, since MRAS expects non-negative normalized values

## Inputs

- Bulk gene expression matrix (gene x sample, human gene symbols as row names, normalized non-negative values, e.g. bulk_test_example)
- Human-GEM model information (13082 metabolic reactions with reaction IDs, GPR gene associations, reversibility, compartments, SUBSYSTEM pathway labels)

## Outputs

- MRAS matrix of 13082 Human-GEM reaction rows x samples, single-sample normalized, keyed by Human-GEM Reaction IDs with no missing or negative scores (e.g. mras_scores.csv)

## How to apply

Load the gene expression matrix (e.g. bulk_test_example) and verify it is a gene x sample matrix with human gene symbols as row names and no negative entries. Map expression onto the Human-GEM model's 13082 reactions via GPR rules: for AND logic (multi-subunit complexes) assign the minimum expression across subunits, for OR logic (isoenzymes) sum isoenzyme contributions, and apply the promiscuity adjustment by dividing each gene's expression by the number of reactions that gene participates in. Then perform single-sample normalization on each sample's reaction scores so the output MRAS matrix is 13082 reactions x samples with no missing or negative values. This MRAS matrix, not raw expression, is the input METAFlux expects for the subsequent FBA (compute_flux with a medium file such as human_blood or cell_medium).

## Related tools

- **METAFlux** (R package whose calculate_reaction_score function computes the single-sample normalized MRAS from gene expression using GPR rules) — https://github.com/KChen-lab/METAFlux
- **R** (Language environment in which the METAFlux library and GPR mapping/normalization are executed)

## Examples

```
library(METAFlux); data("bulk_test_example"); scores <- calculate_reaction_score(bulk_test_example)
```

## Evaluation signals

- Output mras_scores.csv has exactly 13082 reaction rows keyed by Human-GEM Reaction IDs
- One MRAS column per input sample; scores are single-sample normalized within each column
- No missing or negative values anywhere in the MRAS matrix
- Downstream compute_flux accepts the MRAS matrix together with a medium file (human_blood or cell_medium) without key errors
- Gene promiscuity adjustment applied: each gene's expression divided by the number of reactions it participates in before aggregation

## Limitations

- MRAS quality depends on the GPR mapping choices (min for AND complexes, sum for OR isoenzymes, promiscuity division); reactions without GPR gene associations cannot be scored from expression
- No software version number for METAFlux or R is stated in the provided text, so reproducibility pinning is unclear
- The current objective function maximizes the generic biomass reaction (biomass_human from Human1); MRAS is only the reaction-level input, not a flux estimate

## Evidence

- [methods] GPR-based MRAS computation: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
- [methods] reaction score function call: "scores<-calculate_reaction_score(bulk_test_example)"
- [methods] input matrix format requirement: "The gene expression matrix should be gene x sample matrix where row names are **human gene names** (gene symbols)"
- [methods] Human-GEM scale: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [intro] per-sample flux score count: "METAFlux can predict 13,082 reaction flux scores for each bulk sample"
