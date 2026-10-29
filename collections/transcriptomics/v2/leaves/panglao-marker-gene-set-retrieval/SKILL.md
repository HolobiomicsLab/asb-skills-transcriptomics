---
name: panglao-marker-gene-set-retrieval
description: Use when validating Pollock top-100 DWG lists (from captum feature scores)
  against a trusted external marker database, or when quantifying per-cell-state overlap
  percentages with known markers.
when_to_use_negative:
- The task is cell type classification/prediction itself (use Pollock predict mode),
  not marker-based validation of learned features
- No ground-truth marker database mapping exists for the cell states in question,
  so overlap percentages would be undefined
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Pollock
  role: explain mode generates the per-cell-state DWG rankings (captum feature scores)
    that are intersected with PanglaoDB marker sets
  repo: github.com/ding-lab/pollock
- name: captum
  role: feature attribution backend producing the interpretability scores from which
    top-100 DWGs are selected
- name: scanpy
  role: single-cell data handling environment in which the anndata inputs and gene
    lists are processed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:ec1176cf5ab3f7bc348849578cf4eb0a8d3006084b245c2e2126e9ec68e1e4c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pollock
  - captum
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# panglao-marker-gene-set-retrieval

## Summary

Retrieves curated PanglaoDB marker gene sets (release 27_Mar_2020, human-support-filtered) and computes their overlap with Pollock explain-mode DWG lists to validate learned cell-state features against known markers.

## When to use

Use when validating Pollock top-100 DWG lists (from captum feature scores) against a trusted external marker database, or when quantifying per-cell-state overlap percentages with known markers.

## When NOT to use

- The task is cell type classification/prediction itself (use Pollock predict mode), not marker-based validation of learned features
- No ground-truth marker database mapping exists for the cell states in question, so overlap percentages would be undefined

## Inputs

- Top-100 DWG lists per cell state derived from Pollock explain-mode captum feature scores (pretrained module)
- PanglaoDB marker gene files, release 27_Mar_2020, filtered to entries with human support
- Mapping of Pollock cell states to PanglaoDB cell types (direct or macro parent)

## Outputs

- Per-cell-state overlap percentage table (panglao_overlap_table.csv)
- Verification that B, pDC, and Treg states reproduce the reported 25–29% overlap range

## How to apply

First generate top-100 differentially weighted genes (DWGs) per cell state via Pollock explain mode, which uses captum feature scores on the pretrained module. Then download PanglaoDB marker gene files from release 27_Mar_2020, filtered to entries with human support. Map each Pollock cell state (e.g. B cell, pDC, Treg) to its corresponding PanglaoDB cell type; when no direct match exists, fall back to the marker set of the next-most-macro parent cell state. Compute each overlap percentage as the size of the intersection between the state's top-100 DWGs and the matched marker set, divided by 100. Assemble per-state percentages into a table (panglao_overlap_table.csv) and check that B, pDC, and Treg fall in the reported 25–29% range.

## Related tools

- **Pollock** (explain mode generates the per-cell-state DWG rankings (captum feature scores) that are intersected with PanglaoDB marker sets) — github.com/ding-lab/pollock
- **captum** (feature attribution backend producing the interpretability scores from which top-100 DWGs are selected)
- **scanpy** (single-cell data handling environment in which the anndata inputs and gene lists are processed)

## Examples

```
pollock explain from_seurat --module-filepath <module_dir> --explain-filepath cells.rds --background-filepath background.rds --background-sample-size 100 --output-prefix output
```

## Evaluation signals

- Each overlap percentage equals (intersection size between top-100 DWGs and matched PanglaoDB marker set) / 100
- B, pDC, and Treg cell states reproduce overlap values in the reported 25–29% range
- PanglaoDB marker file is the 27_Mar_2020 release filtered to human-support entries
- States lacking a direct PanglaoDB match use the next-most-macro parent cell state's marker set, and the mapping is documented
- Output CSV (panglao_overlap_table.csv) contains one row per immune cell state with a percentage column

## Limitations

- Explain mode can have excessive runtimes for very large numbers of cells; downsample cells and use a background sample size of ~100 (≈15 minutes for ~100 explained cells)
- Overlap is limited to the top-100 DWGs per state, so percentages are capped by this ranking depth
- Cell states without a direct PanglaoDB cell-type match must rely on macro parent marker sets, which may dilute specificity

## Evidence

- [other] PanglaoDB release and human filter: [evidence span withheld — anchor status: not_found]
- [other] Overlap computation rule: [evidence span withheld — anchor status: not_found]
- [abstract] Interpretability scores drive classifications: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [readme] Pollock model architecture grounding the DWG source: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] Explain-mode runtime caveat: "Note: explain mode can have excessive runtimes for very large numbers of cells, so we recommend downsampling the"
