---
name: harmony-benchmark-integration-comparison
description: Use when evaluating a new integration/annotation method's batch-effect
  removal and you need a head-to-head LISI-based comparison against Harmony on the
  same merged datasets.
when_to_use_negative:
- The task is cell-type annotation itself rather than benchmarking integration — Harmony
  produces integrated embeddings, not cell-type labels like SELINA's query_predictions.txt
- Disease-mode SELINA prediction, where fine-tuning is skipped and the paper recommends
  users rely on their own data rather than a shared integration benchmark
- Data are not batch-structured (no platform/reference-query grouping variables),
  making LISI of common vs specific cell groups undefined
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Harmony
  role: Baseline integration method run on the same merged liver data to produce comparable
    integrated embeddings and LISI scores (Figures 3D-3F)
  repo: https://github.com/immunogenomics/harmony
- name: SELINA
  role: Method under comparison; its MADA pre-training and autoencoder fine-tuning
    LISI scores are benchmarked against Harmony's
  repo: https://github.com/SELINA-team/SELINA.py
- name: R
  role: Runtime environment for Harmony (tested on R versions >= 4.2)
- name: Python
  role: Environment for running SELINA (selina train / selina predict) and computing
    LISI scores for the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:51d60990e61e7957cca518e9f0992882c1f9fbd019270231a37387980b25e449
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/immunogenomics/harmony
  tools:
  - Harmony
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# harmony-benchmark-integration-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark baseline for batch integration: re-run the same merged single-cell reference/query data through Harmony and compute matching LISI scores so SELINA's MADA and autoencoder-based batch removal can be quantitatively compared against a established integration method (Figures 3D-3F of the SELINA paper).

## When to use

Use when evaluating a new integration/annotation method's batch-effect removal and you need a head-to-head LISI-based comparison against Harmony on the same merged datasets.

## When NOT to use

- The task is cell-type annotation itself rather than benchmarking integration — Harmony produces integrated embeddings, not cell-type labels like SELINA's query_predictions.txt
- Disease-mode SELINA prediction, where fine-tuning is skipped and the paper recommends users rely on their own data rather than a shared integration benchmark
- Data are not batch-structured (no platform/reference-query grouping variables), making LISI of common vs specific cell groups undefined

## Inputs

- Merged gene-by-cell expression matrix from five liver reference datasets (*_expr.txt)
- Cell metadata with cell type and sequencing platform labels (*_meta.txt)
- Query data with reference/query provenance labels for QR-common and QR-specific groups

## Outputs

- lisi_scores.csv with platform-LISI and reference-query-LISI columns for both SELINA and Harmony
- UMAP PNG plots of the Harmony-integrated embeddings
- Harmony LISI values for Figures 3D-3F comparison

## How to apply

Merge the candidate datasets (in the SELINA paper, five liver reference datasets with paired *_expr.txt and *_meta.txt files) into one combined matrix, exactly as prepared for the method under test. Run Harmony on this same merged data using the immunogenomics/harmony R package (tested on R >= 4.2, installable via devtools::install_github or CRAN; Python users can use harmonypy). Extract Harmony's integrated embeddings and compute the same LISI metrics used for the method under test: platform-LISI on platform-common vs platform-specific cells and reference-query LISI on QR-common vs QR-specific cell groups. Write all values to lisi_scores.csv and render embedding UMAPs to PNG files, so Harmony's LISI pattern (increased LISI for common cells, decreased for specific cells after integration) can be compared directly with SELINA's Figures 3C/3G.

## Related tools

- **Harmony** (Baseline integration method run on the same merged liver data to produce comparable integrated embeddings and LISI scores (Figures 3D-3F)) — https://github.com/immunogenomics/harmony
- **SELINA** (Method under comparison; its MADA pre-training and autoencoder fine-tuning LISI scores are benchmarked against Harmony's) — https://github.com/SELINA-team/SELINA.py
- **R** (Runtime environment for Harmony (tested on R versions >= 4.2))
- **Python** (Environment for running SELINA (selina train / selina predict) and computing LISI scores for the comparison)

## Examples

```
Rscript -e 'library(harmony); integrated <- RunHarmony(seurat_obj, group.by.vars = "platform")'
```

## Evaluation signals

- lisi_scores.csv exists and contains platform-LISI and reference-query-LISI columns for both SELINA and Harmony
- Harmony, like SELINA, shows increased LISI for platform-common / QR-common cells and decreased LISI for platform-specific / QR-specific cells relative to pre-integration
- The same merged input data and cell groupings (platform-common, platform-specific, QR-common, QR-specific) are used for both tools so LISI values are directly comparable
- UMAP PNGs of the Harmony embedding are generated alongside the numeric LISI table, mirroring the SELINA embedding plots

## Limitations

- Harmony is used as a comparison baseline only; the paper's contribution is SELINA's MADA-based batch removal, so Harmony LISI values serve as the reference point (Figures 3D-3F), not an independent result
- Harmony requires R >= 4.2 and its package dependencies (see DESCRIPTION); Python users must fall back on the separate harmonypy package
- LISI comparisons depend on correct platform and reference/query labels in the metadata; mislabeled or missing platform metadata invalidates the common/specific cell group split

## Evidence

- [results] both the feature extractor and cell-type classifier transformations increased LISI scores of platform-common cells and decreased LISI scores of platform-specific cells: [evidence span withheld — anchor status: not_found]
- [methods] Re-run the same merged data through Harmony and compute the matching LISI scores for comparison: [evidence span withheld — anchor status: not_found]
- [readme] Integration of large, complex single-cell datasets with Harmony: [evidence span withheld — anchor status: external_doc]
- [readme] Harmony has been tested on R versions >= 4.2: [evidence span withheld — anchor status: external_doc]
- [methods] verify lisi_scores.csv exists with platform-LISI and reference-query-LISI columns for both SELINA and Harmony: [evidence span withheld — anchor status: not_found]
