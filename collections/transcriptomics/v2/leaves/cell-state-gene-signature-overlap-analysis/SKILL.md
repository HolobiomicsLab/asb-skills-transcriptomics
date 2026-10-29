---
name: cell-state-gene-signature-overlap-analysis
description: Use when validating a learned cell-state classifier (e.g., Pollock explain-mode
  DWGs) against curated marker databases like PanglaoDB, or when asked which cell
  states' feature scores recover known markers.
when_to_use_negative:
- The goal is cell type prediction itself (use Pollock predict mode) rather than validating
  interpretability scores against known markers.
- The comparison target is not a curated marker database but, e.g., a differential
  expression analysis; this skill's denominator and mapping assumptions assume PanglaoDB-style
  marker sets.
- The pretrained module's classes are non-immune tissue states with no direct or macro-parent
  match in the PanglaoDB marker files, making the state-to-marker mapping undefined.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: explain mode generates per-cell-state captum feature scores from which the
    top-100 DWGs are taken
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: provides the feature attribution scores underlying the DWG ranking
- name: scanpy
  role: loads/hosts the .h5ad anndata objects explained in explain mode
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:54bfaba0b4b1bb733e009a76b94f0566e24ee64d646e3939434d05cb93d71782
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - captum
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-state-gene-signature-overlap-analysis

## Summary

Quantifies biological plausibility of Pollock's captum-derived interpretability scores by intersecting the top-100 differentially weighted genes (DWGs) per cell state with PanglaoDB marker gene sets, reporting per-state overlap percentages.

## When to use

Use when validating a learned cell-state classifier (e.g., Pollock explain-mode DWGs) against curated marker databases like PanglaoDB, or when asked which cell states' feature scores recover known markers.

## When NOT to use

- The goal is cell type prediction itself (use Pollock predict mode) rather than validating interpretability scores against known markers.
- The comparison target is not a curated marker database but, e.g., a differential expression analysis; this skill's denominator and mapping assumptions assume PanglaoDB-style marker sets.
- The pretrained module's classes are non-immune tissue states with no direct or macro-parent match in the PanglaoDB marker files, making the state-to-marker mapping undefined.

## Inputs

- Top-100 DWG lists per cell state (derived from Pollock explain-mode captum feature scores on a pretrained module)
- PanglaoDB marker gene files (release 27_Mar_2020), filtered to entries with human support
- Mapping of Pollock cell states to PanglaoDB cell types (with macro-parent fallback assignments)

## Outputs

- panglao_overlap_table.csv containing per-cell-state overlap percentages between top-100 DWGs and PanglaoDB marker sets
- Identification of the cell states with the most marker overlap (B, pDC, Treg at 25-29%)

## How to apply

First obtain the top-100 DWG lists per cell state from Pollock explain mode, which uses captum feature scores on a pretrained module (e.g., scRNAseq_brca_panimmune with immune cell states such as B cell, pDC, Treg). Download the PanglaoDB marker gene files (release 27_Mar_2020) and filter entries to those with human support. Map each Pollock cell state to its corresponding PanglaoDB cell type; when no direct match exists, fall back to the marker set of the next-most-macro parent cell state. Compute the overlap percentage per state as |intersection(top-100 DWGs, PanglaoDB markers)| / 100, then assemble the per-state percentages into panglao_overlap_table.csv and check that B, pDC, and Treg reproduce the reported 25-29% overlap range.

## Related tools

- **Pollock** (explain mode generates per-cell-state captum feature scores from which the top-100 DWGs are taken) — https://github.com/ding-lab/pollock
- **captum** (provides the feature attribution scores underlying the DWG ranking)
- **scanpy** (loads/hosts the .h5ad anndata objects explained in explain mode)

## Examples

```
pollock explain from_scanpy --module-filepath <module_dir> --scanpy-h5ad-filepath <input.h5ad> --explain-filepath <explain.h5ad> --background-filepath <background.h5ad>
```

## Evaluation signals

- B, pDC, and Treg cell states reproduce overlap percentages in the reported 25-29% range.
- Each overlap percentage is computed as intersection size divided by 100 (the DWG list length), so values cannot exceed 100%.
- The per-state results assemble into a complete panglao_overlap_table.csv covering all mapped cell states.
- Every PanglaoDB entry used is from release 27_Mar_2020 and carries human support.
- States lacking a direct PanglaoDB match are resolved via the next-most-macro parent cell state, never left unmapped or dropped silently.

## Limitations

- Explain mode has excessive runtimes for large numbers of cells (~15 minutes for ~100 cells with a background sample size of ~100), so cells should be downsampled.
- When no direct PanglaoDB cell type matches a Pollock cell state, the parent-state fallback introduces coarser marker sets and can inflate or deflate overlap percentages.
- Expression data passed to explain mode must be raw counts (unnormalized); input .h5ad objects with normalized .X are not valid.

## Evidence

- [abstract] provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [methods] An example of explaining a model for a Seurat .RDS object: "An example of explaining a model for a Seurat .RDS object that has cell type labels in a metadata column named 'cell_type'"
- [readme] explain mode can have excessive runtimes for very large numbers of cells: "Note: explain mode can have excessive runtimes for very large numbers of cells, so we recommend downsampling"
- [readme] Number of cells to sample as background samples. The default of 100 cells is sufficient: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
