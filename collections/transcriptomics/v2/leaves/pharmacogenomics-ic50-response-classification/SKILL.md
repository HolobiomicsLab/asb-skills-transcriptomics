---
name: pharmacogenomics-ic50-response-classification
description: Use when raw drug IC50 values from a pharmacogenomics dataset (e.g.,
  GDSC1000 cell-line panel) must be binned into sensitive/resistant/null classes per
  cell line before computing regimen coverage or selecting combination partners.
when_to_use_negative:
- Input is single-cell omics data (scRNAseq target-gene enrichment or CyTOF proteomics)
  rather than bulk pharmacogenomics IC50 values — use the scRNAseq/CyTOF screening
  and validation steps of the secDrug pipeline instead
- The analysis goal is ranking drug combinations by confidence score from already-classified
  response groups rather than producing the classification itself
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_2640
tools:
- name: CombinationDrugMyeloma
  role: R package whose AssignGroup(x, l) function performs the S/R/N classification
    of per-cell-line IC50 values
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Environment in which the GDSC1000 data are loaded, filtered to the cell-line
    panel, and classified with AssignGroup
- name: secDrug
  role: Downstream greedy set-cover algorithm that consumes the classified response
    groups to select secondary drugs
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:98efac4845237d521ad1b30bdd47a4cec8f609e027b99d2eeab4c3a24a5a6243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pharmacogenomics-ic50-response-classification

## Summary

Classifies cell lines from a pharmacogenomics screen (e.g., GDSC1000 IC50 values) into drug-response groups — sensitive (S), resistant (R), or null (N) — using the AssignGroup function of the CombinationDrugMyeloma R package, so downstream set-cover combination algorithms operate on a categorical, comparable response scale rather than raw IC50s.

## When to use

Use when raw drug IC50 values from a pharmacogenomics dataset (e.g., GDSC1000 cell-line panel) must be binned into sensitive/resistant/null classes per cell line before computing regimen coverage or selecting combination partners.

## When NOT to use

- Input is single-cell omics data (scRNAseq target-gene enrichment or CyTOF proteomics) rather than bulk pharmacogenomics IC50 values — use the scRNAseq/CyTOF screening and validation steps of the secDrug pipeline instead
- The analysis goal is ranking drug combinations by confidence score from already-classified response groups rather than producing the classification itself

## Inputs

- GDSC1000 pharmacogenomics dataset (drug IC50 values per cell line) from cancerrxgene.org, loaded in R
- Filtered cell-line panel of computable B-cell cancer / human myeloma cell lines (e.g., 94 computable B-cell lines)
- Backbone drug selection (e.g., proteasome inhibitor) with per-cell-line IC50 values

## Outputs

- Per-cell-line categorical response groups (S = sensitive, R = resistant, N = null) for the backbone PI drug
- Categorized PI response groups suitable as input to the secDrug greedy set-cover algorithm and CombinationDrugs() coverage computation

## How to apply

Download the public GDSC1000 pharmacogenomics dataset from cancerrxgene.org and filter it to the computable B-cell cancer/myeloma cell-line panel (e.g., the 94 computable lines used for coverage computation). Load the data in R and apply the AssignGroup(x, l) function from the CombinationDrugMyeloma package to categorize each cell line's proteasome-inhibitor IC50 as sensitive (S), resistant (R), or null (N); the null group handles cell lines with missing or non-computable response values so they are excluded from coverage denominators rather than misclassified. The resulting S/R/N grouping vector is the required input to the secDrug greedy set-cover algorithm and to CombinationDrugs(dir, file, dest), which computes percent cell-line coverage of PI-backbone regimens. The rationale is that discretizing IC50 into resistance classes makes the set-cover coverage objective well-defined: a regimen 'covers' a line if it kills lines resistant to the backbone drug.

## Related tools

- **CombinationDrugMyeloma** (R package whose AssignGroup(x, l) function performs the S/R/N classification of per-cell-line IC50 values) — github.com/Ujjal-Mukherjee/secDrug
- **R** (Environment in which the GDSC1000 data are loaded, filtered to the cell-line panel, and classified with AssignGroup)
- **secDrug** (Downstream greedy set-cover algorithm that consumes the classified response groups to select secondary drugs) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
groups <- CombinationDrugMyeloma::AssignGroup(pi_ic50, cutoffs); CombinationDrugMyeloma::CombinationDrugs(dir, file, dest)
```

## Evaluation signals

- Every cell line in the panel receives exactly one of the labels S, R, or N — no raw IC50 values remain unclassified
- PI-only coverage computed from the groups reproduces the published 33.0% baseline on the 94 computable B-cell lines
- Downstream top regimen PI+FK866+17-AAG coverage matches published values: 72.2% with 2 secDrugs and 82.5% with 3 secDrugs
- Coverage percentages are computed against the same denominator of computable cell lines used for grouping (null/N lines excluded consistently)

## Limitations

- Classification is defined only for cell lines with computable IC50 values; lines with null response (N) are excluded from coverage and can change coverage percentages if the panel changes
- The published pipeline currently supports two-drug (PI backbone + secDrug) combinations; model systems for more than two secDrugs were not yet developed
- Broader HMCL panels representing refractory and acquired-resistant states have not yet been tested, so group labels may not generalize to wider panels

## Evidence

- [methods] AssignGroup categorization step: [evidence span withheld — anchor status: not_found]
- [abstract] greedy algorithm prediction: "a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma"
- [intro] HMCL panel rationale: "We have compiled a panel of >70 human myeloma cell lines (HMCLs) representing the broad spectrum of biological and genetic heterogeneity of myeloma patients"
- [discussion] wider panel caveat: "builds a strong case to test these drugs as secDrug combination regimens in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines."
