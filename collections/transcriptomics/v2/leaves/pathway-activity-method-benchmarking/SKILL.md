---
name: pathway-activity-method-benchmarking
description: Use when you need to quantitatively evaluate whether a pathway activity
  method outperforms alternatives (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) on controlled
  single-cell simulations with a known ground-truth perturbed pathway.
when_to_use_negative:
- The goal is biological discovery on real patient data (e.g. COVID-19 peripheral
  blood atlas comparisons) rather than controlled method evaluation with a known implanted
  pathway.
- The competing tools cannot be run on the identical simulated input — a fair sensitivity
  comparison requires all methods to see the same two-population sample pairs.
- Only a single simulation scenario is available; sensitivity (fraction of scenarios
  with detection) cannot be meaningfully estimated from one point.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCPA
  role: Method under test; run via compare_pathways() on each simulated two-population
    sample pair to detect the implanted pathway via multivariate distribution testing
  repo: https://github.com/jackbibby1/SCPA
- name: Splatter
  role: Generates simulated two-population scRNA-seq datasets via splatEstimate()
    and splatSimulate(), with the 200-gene perturbed pathway implanted among ~17,000
    background genes
- name: fGSEA
  role: Competitor enrichment-based pathway method run on the identical simulated
    datasets for sensitivity comparison
- name: iDEA
  role: Competitor pathway analysis method run on the identical simulated datasets
    for sensitivity comparison
- name: GSVA
  role: Competitor pathway activity method run on the identical simulated datasets
    for sensitivity comparison
- name: AUCell
  role: Competitor pathway activity method run on the identical simulated datasets
    for sensitivity comparison
- name: Vision
  role: Competitor pathway activity method run on the identical simulated datasets
    for sensitivity comparison
- name: ssGSEA
  role: Competitor enrichment method run on the identical simulated datasets for sensitivity
    comparison
- name: Seurat
  role: Provides the reference single-cell object format from which simulation parameters
    are estimated and expression is extracted (e.g. seurat_extract())
- name: msigdbr
  role: Sources pathway gene sets (e.g. Hallmark) used as the defined gene sets when
    running compare_pathways()
- name: dplyr
  role: Data manipulation for assembling the per-method per-scenario sensitivity results
- name: ggplot2
  role: Visualization of method-by-scenario sensitivity results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:e6e47d066b04881a52ad0755efdf8a7ef2e143e3b159b7ae04f2f7c8d22a2a87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - AUCell
  - GSVA
  - R
  - SCPA
  - Seurat
  - Splatter
  - Vision
  - dplyr
  - fGSEA
  - ggplot2
  - iDEA
  - msigdbr
  - ssGSEA
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# pathway-activity-method-benchmarking

## Summary

Benchmarking protocol for comparing the sensitivity of a single-cell pathway analysis method (here SCPA) against competitor tools on Splatter-simulated scRNA-seq datasets with an implanted 200-gene perturbed pathway, quantifying detection sensitivity per method per scenario.

## When to use

Use when you need to quantitatively evaluate whether a pathway activity method outperforms alternatives (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) on controlled single-cell simulations with a known ground-truth perturbed pathway.

## When NOT to use

- The goal is biological discovery on real patient data (e.g. COVID-19 peripheral blood atlas comparisons) rather than controlled method evaluation with a known implanted pathway.
- The competing tools cannot be run on the identical simulated input — a fair sensitivity comparison requires all methods to see the same two-population sample pairs.
- Only a single simulation scenario is available; sensitivity (fraction of scenarios with detection) cannot be meaningfully estimated from one point.

## Inputs

- Reference single-cell dataset (e.g. Seurat object / .rds) for splatEstimate() parameter estimation
- Splatter simulation parameters (from splatEstimate())
- 200-gene target pathway gene set matrix for perturbation implantation
- ~17,000-gene simulated expression matrices for two populations per scenario
- Pathway gene set files (MSigDB Hallmark/canonical/GO/regulatory pathways, csv or gmt)

## Outputs

- sensitivity_benchmark.csv with one sensitivity entry per method per simulated scenario
- Per-scenario significance calls for SCPA and each competitor (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA)
- Optional ggplot2 sensitivity comparison plots

## How to apply

First estimate simulation parameters from a reference single-cell dataset with splatEstimate() and generate two-population comparisons with splatSimulate(), embedding a 200-gene pathway matrix carrying the implanted perturbation among ~17,000 background genes. Sweep simulation scenarios by varying perturbation strength to build a panel of simulated sample pairs with known ground truth. Run SCPA's compare_pathways() on each simulated pair using defined gene sets (e.g. MSigDB Hallmark via msigdbr and format_pathways()), and run fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA on the identical datasets. For each method, compute sensitivity as the fraction of scenarios in which the implanted 200-gene pathway is detected as significant. Judge correctness by verifying one sensitivity entry per method per scenario and that SCPA's distribution-based qval statistic recovers the implanted pathway more often than enrichment-based competitors.

## Related tools

- **SCPA** (Method under test; run via compare_pathways() on each simulated two-population sample pair to detect the implanted pathway via multivariate distribution testing) — https://github.com/jackbibby1/SCPA
- **Splatter** (Generates simulated two-population scRNA-seq datasets via splatEstimate() and splatSimulate(), with the 200-gene perturbed pathway implanted among ~17,000 background genes)
- **fGSEA** (Competitor enrichment-based pathway method run on the identical simulated datasets for sensitivity comparison)
- **iDEA** (Competitor pathway analysis method run on the identical simulated datasets for sensitivity comparison)
- **GSVA** (Competitor pathway activity method run on the identical simulated datasets for sensitivity comparison)
- **AUCell** (Competitor pathway activity method run on the identical simulated datasets for sensitivity comparison)
- **Vision** (Competitor pathway activity method run on the identical simulated datasets for sensitivity comparison)
- **ssGSEA** (Competitor enrichment method run on the identical simulated datasets for sensitivity comparison)
- **Seurat** (Provides the reference single-cell object format from which simulation parameters are estimated and expression is extracted (e.g. seurat_extract()))
- **msigdbr** (Sources pathway gene sets (e.g. Hallmark) used as the defined gene sets when running compare_pathways())
- **dplyr** (Data manipulation for assembling the per-method per-scenario sensitivity results)
- **ggplot2** (Visualization of method-by-scenario sensitivity results)

## Examples

```
library(SCPA); library(Seurat); library(msigdbr); params <- splatEstimate(ref_counts); sim <- splatSimulate(params, nGenes = 17000, groupCells = c(500, 500)); out <- compare_pathways(samples, pathways)
```

## Evaluation signals

- sensitivity_benchmark.csv contains one sensitivity entry per method per simulated scenario (complete method x scenario grid)
- SCPA recovers the implanted 200-gene pathway in a greater fraction of scenarios than fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA
- Each simulated scenario was generated with the same ~17,000 background genes and a 200-gene implanted pathway so all methods see identical input
- SCPA significance calls are based on the qval as the primary statistic, with fold change/enrichment used only as secondary information
- Simulation parameters were derived from a real reference dataset via splatEstimate() rather than arbitrary values

## Limitations

- Sensitivity estimates depend on the chosen range of simulated perturbation strengths/parameters; results may not transfer to real biological perturbations.
- Benchmark is limited to the six competitors tested (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) and two-population comparisons; multisample settings are not evaluated.
- SCPA's qval can be large even without fold change/enrichment (e.g. arachidonic acid metabolism), so detection calls must be interpreted with qval as the primary statistic.

## Evidence

- [readme] SCPA handles both enrichment and enrichment-independent change: [evidence span withheld — anchor status: external_doc]
