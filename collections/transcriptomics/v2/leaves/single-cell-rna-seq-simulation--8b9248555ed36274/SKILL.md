---
name: single-cell-rna-seq-simulation
description: 'Use when benchmarking pathway-analysis methods on single-cell data:
  you need controlled simulated two-population comparisons with a known implanted
  target pathway so detection sensitivity can be measured as a fraction of recovered
  scenarios.'
when_to_use_negative:
- You have real biological two-condition data and want biological interpretation of
  pathway dysregulation (e.g., COVID-19 monocyte analysis) — run SCPA directly on
  the data rather than on simulations
- The goal is differential expression or gene-level testing rather than method benchmarking
  of pathway-level sensitivity
- Input is a single population/timepoint without a two-group contrast to implant the
  perturbation across
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: 'Simulates the two-population single-cell datasets: splatEstimate() fits parameters
    from the reference data and splatSimulate() generates the scenario panel'
- name: SCPA
  role: Method under test — compare_pathways() scores pathway activity as a change
    in multivariate distribution on each simulated pair
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Provides the single-cell object format and seurat_extract() to obtain expression
    matrices for analysis
- name: msigdbr
  role: Supplies MSigDB Hallmark and other pathway gene sets used as the gene-set
    universe in the simulation benchmark
- name: fGSEA
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: iDEA
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: GSVA
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: AUCell
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: Vision
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: ssGSEA
  role: Competitor pathway method evaluated on the identical simulated datasets
- name: dplyr
  role: Data wrangling of method-by-scenario sensitivity results before export
- name: ggplot2
  role: Visualization of sensitivity comparison across methods and scenarios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:1f26740949df88b03d3a91f6dd93518e6069625b348fbdc33df342040ff61c43
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# single-cell-rna-seq-simulation

## Summary

Generate realistic simulated two-population single-cell RNA-seq benchmark datasets with Splatter — estimating parameters from a reference dataset (~17,000 background genes) and implanting a perturbed 200-gene target pathway — to quantitatively compare the sensitivity of SCPA against competing pathway tools (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA).

## When to use

Use when benchmarking pathway-analysis methods on single-cell data: you need controlled simulated two-population comparisons with a known implanted target pathway so detection sensitivity can be measured as a fraction of recovered scenarios.

## When NOT to use

- You have real biological two-condition data and want biological interpretation of pathway dysregulation (e.g., COVID-19 monocyte analysis) — run SCPA directly on the data rather than on simulations
- The goal is differential expression or gene-level testing rather than method benchmarking of pathway-level sensitivity
- Input is a single population/timepoint without a two-group contrast to implant the perturbation across

## Inputs

- Reference single-cell dataset (e.g., Seurat object or SingleCellExperiment) for splatEstimate() parameter estimation
- Target pathway gene set (200-gene pathway matrix) with perturbation to implant
- Merged pathway gene set file (e.g., combined_metabolic_pathways.csv or h_k_r_go_pid_reg_wik.csv from MSigDB Hallmark/GO/regulatory sets)

## Outputs

- Panel of simulated two-population scRNA-seq sample pairs across varied perturbation scenarios
- sensitivity_benchmark.csv — one sensitivity entry per method per simulated scenario

## How to apply

First estimate simulation parameters from a reference single-cell dataset using splatEstimate(), then generate two simulated populations with splatSimulate(), keeping ~17,000 background genes plus a separate 200-gene pathway matrix into which the perturbation is implanted. Sweep perturbation strength/parameters across scenarios to build a panel of simulated two-population comparisons on which all methods are evaluated identically. Run SCPA's compare_pathways() on each simulated sample pair with the defined gene sets, then run fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA on the same data. Score each method by sensitivity — the fraction of scenarios in which the implanted 200-gene pathway is called significant — using SCPA's qval as the primary statistic, since the paper recommends fold change/enrichment only as secondary informative values. Write the method-by-scenario sensitivity table to sensitivity_benchmark.csv.

## Related tools

- **Splatter** (Simulates the two-population single-cell datasets: splatEstimate() fits parameters from the reference data and splatSimulate() generates the scenario panel)
- **SCPA** (Method under test — compare_pathways() scores pathway activity as a change in multivariate distribution on each simulated pair) — https://github.com/jackbibby1/SCPA
- **Seurat** (Provides the single-cell object format and seurat_extract() to obtain expression matrices for analysis)
- **msigdbr** (Supplies MSigDB Hallmark and other pathway gene sets used as the gene-set universe in the simulation benchmark)
- **fGSEA** (Competitor pathway method evaluated on the identical simulated datasets)
- **iDEA** (Competitor pathway method evaluated on the identical simulated datasets)
- **GSVA** (Competitor pathway method evaluated on the identical simulated datasets)
- **AUCell** (Competitor pathway method evaluated on the identical simulated datasets)
- **Vision** (Competitor pathway method evaluated on the identical simulated datasets)
- **ssGSEA** (Competitor pathway method evaluated on the identical simulated datasets)
- **dplyr** (Data wrangling of method-by-scenario sensitivity results before export)
- **ggplot2** (Visualization of sensitivity comparison across methods and scenarios)

## Examples

```
params <- splatEstimate(reference_sce); sim <- splatSimulate(params, method = "groups", nGenes = 17200, batchCells = c(n1, n2)); scpa_out <- compare_pathways(samples, pathways)
```

## Evaluation signals

- sensitivity_benchmark.csv contains exactly one sensitivity entry per method per simulated scenario (methods × scenarios complete grid)
- SCPA shows greater sensitivity (higher fraction of scenarios where the implanted 200-gene pathway is recovered as significant) than fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA across the simulated scenario range
- Every simulated pair carries ~17,000 background genes plus the 200-gene implanted pathway, and the implanted pathway is present in the gene-set input to all methods identically
- Ranking of methods is stable when perturbation strength is swept, indicating the sensitivity ordering is not an artifact of a single scenario
- SCPA significance is judged by qval as the primary statistic, not fold change, consistent with the paper's interpretation guidance

## Limitations

- Sensitivity conclusions are simulation-based; Splatter simulations may not capture all real data structure, and parameter estimation from a single reference dataset shapes the scenario space
- Competitor results depend on each method's own significance threshold being applied consistently across the identical simulated inputs
- SCPA (v1.6) auto-detects Seurat version, but installation can require manual dependency fixes (e.g., crossmatch 1.3.1 and multicross 2.1.0), as noted in the README issues

## Evidence

- [results] In simulations, SCPA detected perturbed pathways with greater sensitivity than the commonly used pathway analysis tools (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) across a range of simulated scenarios.: [evidence span withheld — anchor status: not_found]
- [readme] It's a different approach to pathway analysis that defines pathway activity as a change in multivariate distribution of a given pathway across conditions, rather than enrichment or over representation of genes.: [evidence span withheld — anchor status: external_doc]
