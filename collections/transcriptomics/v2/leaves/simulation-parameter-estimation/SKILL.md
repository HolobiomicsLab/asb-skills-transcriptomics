---
name: simulation-parameter-estimation
description: Use when you need sensitivity benchmarks for a pathway or gene-set method
  on synthetic scRNA-seq data with a known implanted target pathway, and ground-truth
  recovery can be scored per scenario.
when_to_use_negative:
- The question is real-data pathway analysis (e.g. comparing COVID-19 vs healthy peripheral
  blood) rather than synthetic benchmarking with a known implanted pathway
- The input is already a scored comparison of pathway activity with no simulation
  component and only needs interpretation (qval as primary statistic, fold change/enrichment
  as secondary)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Splatter
  role: Estimate simulation parameters from reference data (splatEstimate()) and generate
    simulated two-population scRNA-seq datasets with implanted pathway perturbations
    (splatSimulate())
- name: SCPA
  role: 'Method under test: run compare_pathways() on each simulated sample pair to
    detect the implanted pathway'
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Provide/reference the single-cell object format from which simulation parameters
    and expression matrices are derived
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:2d62e47b98097f3c73391a8231d056251d71ed5fa11ca36a1fefc10dfeeaec6d
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
  - ssGSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# simulation-parameter-estimation

## Summary

Estimate simulation parameters from a reference single-cell dataset with Splatter's splatEstimate() and generate controlled two-population simulations (~17,000 background genes plus an implanted 200-gene perturbed pathway) to benchmark pathway analysis tools such as SCPA against fGSEA, GSVA, AUCell and others.

## When to use

Use when you need sensitivity benchmarks for a pathway or gene-set method on synthetic scRNA-seq data with a known implanted target pathway, and ground-truth recovery can be scored per scenario.

## When NOT to use

- The question is real-data pathway analysis (e.g. comparing COVID-19 vs healthy peripheral blood) rather than synthetic benchmarking with a known implanted pathway
- The input is already a scored comparison of pathway activity with no simulation component and only needs interpretation (qval as primary statistic, fold change/enrichment as secondary)

## Inputs

- Reference single-cell expression dataset (e.g. Seurat object or count matrix) for parameter estimation
- Target pathway gene set (200-gene pathway matrix with implanted perturbation)
- Simulation scenario grid (varying perturbation strength/parameters)

## Outputs

- Simulated two-population scRNA-seq datasets per scenario
- sensitivity_benchmark.csv with one sensitivity entry per method per simulated scenario

## How to apply

First estimate simulation parameters from a reference single-cell dataset using Splatter's splatEstimate(), then generate two simulated populations with splatSimulate(), building a ~17,000-gene background with a 200-gene pathway matrix carrying the implanted perturbation. Sweep simulation scenarios by varying perturbation strength/parameters to create a panel of two-population comparisons so sensitivity can be measured as a fraction across scenarios. Run SCPA's compare_pathways() on each simulated sample pair to record whether the implanted pathway is recovered, then run competitors (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) on the identical datasets. Compute sensitivity per method per scenario and write the results to sensitivity_benchmark.csv; validation requires one sensitivity entry per method per scenario and SCPA showing effectiveness over the commonly used methods.

## Related tools

- **Splatter** (Estimate simulation parameters from reference data (splatEstimate()) and generate simulated two-population scRNA-seq datasets with implanted pathway perturbations (splatSimulate()))
- **SCPA** (Method under test: run compare_pathways() on each simulated sample pair to detect the implanted pathway) — https://github.com/jackbibby1/SCPA
- **Seurat** (Provide/reference the single-cell object format from which simulation parameters and expression matrices are derived)

## Examples

```
library(Splatter); params <- splatEstimate(ref_counts); sim <- splatSimulate(params, method = "groups", nGenes = 17000)
```

## Evaluation signals

- sensitivity_benchmark.csv contains one sensitivity entry per method per simulated scenario
- Each simulated dataset carries ~17,000 background genes plus the 200-gene implanted pathway matrix
- SCPA recovers the implanted pathway in at least as many scenarios as every competitor (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA)
- Sensitivity is computed identically for all methods as the fraction of scenarios in which the implanted pathway is detected as significant
- All methods are evaluated on identical simulated datasets per scenario (paired comparison, not independent runs)

## Limitations

- Results reflect Splatter-simulated data with a single 200-gene implanted pathway; real perturbations may be subtler or polygenic
- Sensitivity depends on the scenario grid chosen; conclusions only hold across the simulated range of perturbation strengths tested

## Evidence

- [methods] Estimate simulation parameters from a reference single-cell dataset with splatEstimate() and generate two simulated populations using splatSimulate(), with ~17,000 background genes and a 200-gene pathway matrix: [evidence span withheld — anchor status: not_found]
- [results] In simulations, SCPA detected perturbed pathways with greater sensitivity than the commonly used pathway analysis tools (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA): [evidence span withheld — anchor status: not_found]
