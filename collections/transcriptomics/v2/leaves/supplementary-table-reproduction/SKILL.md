---
name: supplementary-table-reproduction
description: Use when a paper reports a numeric summary table (e.g. marker-overlap
  percentages) and the goal is to independently regenerate those values from source
  data rather than trust the reported numbers.
when_to_use_negative:
- The input is already a computed overlap/summary table — no re-derivation is needed
- The DWG scores were not produced by Pollock explain-mode captum feature scores on
  the pretrained module (different gene-ranking provenance invalidates the comparison)
- The target markers lack human support and the PanglaoDB human-filtered release cannot
  be applied
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: Pollock
  role: Source of the pretrained module and explain mode that produce the DWG feature
    scores being compared to marker sets
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: Provides the feature attribution scores behind explain-mode DWG gene rankings
- name: scanpy
  role: Handles single-cell data loading and preprocessing used with Pollock workflows
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:9dad316e51da628af58dd48672de0388e729f84fcdf1eee3c535823f4b715e4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - captum
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# supplementary-table-reproduction

## Summary

Reproduces a paper-reported summary table — here, the percentage overlap between Pollock's top-100 differentially weighted genes (DWGs) per immune cell state and PanglaoDB marker genes — by re-deriving each cell from primary data and writing the result to panglao_overlap_table.csv.

## When to use

Use when a paper reports a numeric summary table (e.g. marker-overlap percentages) and the goal is to independently regenerate those values from source data rather than trust the reported numbers.

## When NOT to use

- The input is already a computed overlap/summary table — no re-derivation is needed
- The DWG scores were not produced by Pollock explain-mode captum feature scores on the pretrained module (different gene-ranking provenance invalidates the comparison)
- The target markers lack human support and the PanglaoDB human-filtered release cannot be applied

## Inputs

- Top-100 DWG lists per immune cell state from Pollock explain-mode captum feature scores on the pretrained module
- PanglaoDB marker gene files, release 27_Mar_2020, filtered to entries with human support
- Cell-state to PanglaoDB cell-type mapping (with fallback to next-most-macro parent cell state)

## Outputs

- panglao_overlap_table.csv with per-cell-state overlap percentages
- Verification that B, pDC, and Treg states reproduce the reported 25–29% overlap range

## How to apply

Load the top-100 DWG lists per immune cell state, derived from Pollock explain-mode captum feature scores run on the pretrained module (e.g. scRNAseq_brca_panimmune for immune cell states). Download the PanglaoDB marker gene files (release 27_Mar_2020, filtered to entries with human support). Map each cell state to its corresponding PanglaoDB cell type, and when no direct match exists, assign the marker set of the next-most-macro parent cell state. Compute each state's overlap percentage as the intersection size between its top-100 DWGs and the matched PanglaoDB marker set divided by 100, then verify that B, pDC, and Treg states fall in the reported 25–29% range before writing panglao_overlap_table.csv.

## Related tools

- **Pollock** (Source of the pretrained module and explain mode that produce the DWG feature scores being compared to marker sets) — https://github.com/ding-lab/pollock
- **captum** (Provides the feature attribution scores behind explain-mode DWG gene rankings)
- **scanpy** (Handles single-cell data loading and preprocessing used with Pollock workflows)

## Examples

```
pollock explain from_scanpy --module-filepath <pretrained_module> --explain-filepath <predicted.h5ad> --background-filepath <background.h5ad> --background-sample-size 100
```

## Evaluation signals

- B, pDC, and Treg cell states reproduce overlap percentages within the reported 25–29% range
- Each cell state's percentage equals (|top-100 DWGs ∩ matched PanglaoDB marker set|) / 100
- Every cell state has a PanglaoDB mapping, either direct or via the next-most-macro parent cell state
- PanglaoDB release is 27_Mar_2020 and entries are filtered to those with human support
- Output file panglao_overlap_table.csv is produced with one row per immune cell state

## Limitations

- Cell states without a direct PanglaoDB cell-type match must inherit the marker set of a parent state, which can dilute or inflate overlap percentages
- Explain mode has high runtime for large cell numbers — approximately 15 minutes for ~100 cells with a background sample size of ~100
- Overlap is capped by the fixed top-100 DWG window and the chosen PanglaoDB release; other releases or thresholds will change the percentages

## Evidence

- [other] 25–29% of PanglaoDB marker genes are present in the top DWGs for those cell states.: [evidence span withheld — anchor status: not_found]
- [methods] explain mode: model explanation with background sampling: [evidence span withheld — anchor status: not_found]
- [readme] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz"
- [readme] scRNAseq_brca_panimmune (trained with specific immune cell state breast cancer dataset): "scRNAseq_brca_panimmune (trained with specific immune cell state breast cancer dataset)"
