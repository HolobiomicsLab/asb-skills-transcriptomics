---
name: signature-score-enrichment-analysis
description: Use when spatial transcriptomics data (expression matrix plus spot coordinates)
  should be scanned for spatially clustered TME gene signatures — e.g. detecting TLSs
  or tumor capsules — in R with scCancer2.
when_to_use_negative:
- Input is bulk or single-cell (non-spatial) scRNA-seq data with no spot coordinates
  — the spatial autocorrelation tests require a spatial map
- You need cell-level TME subtype annotation rather than spatial structure localization
  — scCancer2's predCellType / runCellSubtypeClassify modules cover that task instead
- The gene signatures of interest have no expected spatial pattern (e.g. housekeeping
  signatures), since a global Moran's I below 0.5 short-circuits the analysis
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCancer2
  role: R package providing the spatial module that computes per-spot signature enrichment
    scores and Global/Local Moran's I statistics on the spatial transcriptomics map
  repo: https://doi.org/10.5281/zenodo.10477296
- name: R
  role: Implementation language for the scCancer2 spatial structure detection workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:d0f83aad356f639f3a5654eeed611cae81f279f2cd598d5c33c2aebd1bc286c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Signature score enrichment analysis

## Summary

Computes per-spot enrichment scores for gene signatures across a spatial transcriptomics map in scCancer2, then tests each signature for spatial structure using Global Moran's I (threshold > 0.5) and localizes significant structures with Local Moran's I. This is the mechanism scCancer2 uses to detect and position TME features such as TLSs and tumor capsules.

## When to use

Use when spatial transcriptomics data (expression matrix plus spot coordinates) should be scanned for spatially clustered TME gene signatures — e.g. detecting TLSs or tumor capsules — in R with scCancer2.

## When NOT to use

- Input is bulk or single-cell (non-spatial) scRNA-seq data with no spot coordinates — the spatial autocorrelation tests require a spatial map
- You need cell-level TME subtype annotation rather than spatial structure localization — scCancer2's predCellType / runCellSubtypeClassify modules cover that task instead
- The gene signatures of interest have no expected spatial pattern (e.g. housekeeping signatures), since a global Moran's I below 0.5 short-circuits the analysis

## Inputs

- Spatial transcriptomics expression matrix (spot-by-gene)
- Spot spatial coordinates
- Gene signature definitions from the scCancer2 spatial module input files

## Outputs

- Per-signature Global Moran's I values
- Filtered per-spot Local Moran's I labels and adjusted P-values written to spatial_structure_results.csv

## How to apply

Load the spatial transcriptomic expression matrix together with spot coordinates and the gene signature definitions from the scCancer2 spatial module input files. Compute per-spot enrichment scores for each gene signature across the spatial map. For each signature, calculate Global Moran's I using inverse-square distance weights between spot pairs and declare the signature spatially clustered when GMI > 0.5. For signatures passing the global test, compute Local Moran's I statistics at each spot to localize hotspot and coldspot regions; correct Local Moran's I P-values with z-score-based correction followed by FDR adjustment, and apply the z-score-based false-positive filtering strategy to retain only robustly significant spots. The two-stage design is intentional: the global statistic declares whether a sample possesses distinct spatial structures, and the local statistic determines their position.

## Related tools

- **scCancer2** (R package providing the spatial module that computes per-spot signature enrichment scores and Global/Local Moran's I statistics on the spatial transcriptomics map) — https://doi.org/10.5281/zenodo.10477296
- **R** (Implementation language for the scCancer2 spatial structure detection workflow)

## Examples

```
suppressMessages(library(scCancer))
```

## Evaluation signals

- Per-signature Global Moran's I values are reported and only signatures with GMI > 0.5 are declared spatially structured and passed to Local Moran's I
- Local Moran's I P-values undergo z-score-based correction followed by FDR adjustment before spot-level significance calls
- The z-score-based false-positive filtering strategy must be applied so that only robustly significant spots retain hotspot/coldspot labels
- Output file spatial_structure_results.csv contains per-signature GMI values and the filtered per-spot Local Moran's I labels and adjusted P-values

## Limitations

- The Global Moran's I cutoff of 0.5 is a fixed declaration threshold; signatures with GMI ≤ 0.5 receive no localization analysis
- No software version numbers are stated for R, scCancer2, or the spatial module dependencies in the methods
- Distance weighting depends on inverse-square distances between spot pairs, so results are sensitive to spot coordinate quality and platform geometry

## Evidence

- [abstract] scCancer2 includes a module to process spatial transcriptomic data and analyze TME spatial features: "Thirdly, scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME."
- [other] Two-stage Global/Local Moran's I detection mechanism: [evidence span withheld — anchor status: not_found]
- [methods] scCancer2 spatial workflow runs in R via the scCancer package: "suppressMessages(library(scCancer))
suppressMessages(library(dplyr))"
- [readme] scCancer toolkit updated with spatial transcriptome support: [evidence span withheld — anchor status: external_doc]
