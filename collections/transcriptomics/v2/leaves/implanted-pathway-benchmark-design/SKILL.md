---
name: implanted-pathway-benchmark-design
description: Use when comparing sensitivity of single-cell pathway analysis methods
  against a ground-truth perturbation, i.e. the analysis goal is a controlled simulation
  where the true significant pathway is known by construction.
when_to_use_negative:
- The goal is biological interpretation of a real dataset (e.g. T cell activation
  dynamics) rather than method-level sensitivity benchmarking — use SCPA's compare_pathways
  directly on the data instead of simulating
- No ground truth exists for the significant pathway, as in a purely exploratory analysis
  where sensitivity cannot be computed
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: SplatEstimate()/splatSimulate() generate the simulated two-population datasets
    with the implanted 200-gene pathway
- name: SCPA
  role: compare_pathways() is run on each simulated sample pair to recover the implanted
    pathway
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Single-cell data container/assay handling for the reference dataset used in
    simulation parameter estimation
- name: msigdbr
  role: Supplies pathway gene sets used as input to compare_pathways
- name: fGSEA
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: iDEA
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: GSVA
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: AUCell
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: Vision
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: ssGSEA
  role: Competitor pathway method benchmarked against SCPA on the identical simulated
    datasets
- name: dplyr
  role: Result table manipulation when assembling the method-by-scenario sensitivity
    table
- name: ggplot2
  role: Visualization of sensitivity results across methods and scenarios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:be4e6d5e1f169b87b91a2fb6c9c605c2b750d71c78bf2455e252063f94140208
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# Implanted-pathway benchmark design for single-cell pathway tools

## Summary

Design a Splatter-based sensitivity benchmark in which a 200-gene target pathway is implanted into simulated single-cell populations over a panel of ~17,000 background genes, then measure how often SCPA and competitor tools (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) recover the implanted pathway.

## When to use

Use when comparing sensitivity of single-cell pathway analysis methods against a ground-truth perturbation, i.e. the analysis goal is a controlled simulation where the true significant pathway is known by construction.

## When NOT to use

- The goal is biological interpretation of a real dataset (e.g. T cell activation dynamics) rather than method-level sensitivity benchmarking — use SCPA's compare_pathways directly on the data instead of simulating
- No ground truth exists for the significant pathway, as in a purely exploratory analysis where sensitivity cannot be computed

## Inputs

- Reference single-cell dataset (e.g. Seurat object or expression matrix) for splatEstimate() parameter estimation
- Splatter simulation parameters (SplatParams object from splatEstimate())
- A 200-gene target pathway matrix implanted as the perturbation signal
- Pathway gene sets (e.g. MSigDB-derived csv via msigdbr / format_pathways)

## Outputs

- Panel of simulated two-population single-cell datasets spanning perturbation scenarios
- Per-method detection calls for the implanted pathway in every scenario
- Sensitivity_benchmark.csv: one sensitivity entry per method per simulated scenario

## How to apply

First estimate simulation parameters from a reference single-cell dataset with splatEstimate() and generate two simulated populations with splatSimulate(), including ~17,000 background genes plus a 200-gene pathway matrix carrying the implanted perturbation. Sweep perturbation strength/parameters to build a panel of simulated two-population comparison scenarios. On each simulated pair, run SCPA's compare_pathways() with the defined gene sets and record whether the implanted pathway is recovered, then run fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA on the identical datasets. Compute sensitivity as the fraction of scenarios in which the implanted 200-gene pathway is called significant, and write the method-by-scenario table to sensitivity_benchmark.csv. The rationale is that simulation provides a known ground truth, letting you quantify which method — SCPA, which models pathway activity as a change in multivariate distribution rather than enrichment — detects perturbed pathways with the greatest sensitivity across scenarios.

## Related tools

- **Splatter** (SplatEstimate()/splatSimulate() generate the simulated two-population datasets with the implanted 200-gene pathway)
- **SCPA** (compare_pathways() is run on each simulated sample pair to recover the implanted pathway) — https://github.com/jackbibby1/SCPA
- **Seurat** (Single-cell data container/assay handling for the reference dataset used in simulation parameter estimation)
- **msigdbr** (Supplies pathway gene sets used as input to compare_pathways)
- **fGSEA** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **iDEA** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **GSVA** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **AUCell** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **Vision** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **ssGSEA** (Competitor pathway method benchmarked against SCPA on the identical simulated datasets)
- **dplyr** (Result table manipulation when assembling the method-by-scenario sensitivity table)
- **ggplot2** (Visualization of sensitivity results across methods and scenarios)

## Examples

```
library(SCPA); params <- splatEstimate(ref_counts); sim <- splatSimulate(params, nCells = 500); scpa_out <- compare_pathways(samples, pathways); sens <- sensitivity_by_method(scpa_out); write.csv(sens, "sensitivity_benchmark.csv")
```

## Evaluation signals

- sensitivity_benchmark.csv contains one sensitivity entry per method per simulated scenario (no missing method-scenario combinations)
- Every scenario in the panel was generated with the same ~17,000 background genes and the same 200-gene implanted pathway, varying only perturbation strength/parameters
- SCPA recovers the implanted pathway in a greater fraction of scenarios than fGSEA, iDEA, GSVA, AUCell, Vision, and ssGSEA, matching the paper's finding that 'SCPA detected perturbed pathways with greater sensitivity'
- Sensitivity values fall in [0, 1] and are computed as the fraction of scenarios in which each method calls the implanted 200-gene pathway significant
- All competitor methods were run on byte-identical simulated datasets to SCPA, ensuring a controlled comparison

## Limitations

- Results depend on Splatter's simulation model and its parameter estimates from the reference dataset; conclusions are about simulated scenarios, not arbitrary real data
- Sensitivity here is defined narrowly as recovery of a single implanted 200-gene pathway; other perturbation structures (multi-pathway, gradient shifts) are not covered by this benchmark
- SCPA's qval should be interpreted carefully — the paper notes pathways can show large qvals without fold change/enrichment (e.g. arachidonic acid metabolism), so significance alone is not an effect-size statement

## Evidence

- [abstract] SCPA demonstrates effectiveness over commonly used pathway analysis methods: "We demonstrate the effectiveness of SCPA over commonly used methods"
- [methods] compare_pathways is the core call for pathway comparison: "To compare these pathways, we can just run the compare_pathways function"
- [readme] SCPA defines pathway activity as multivariate distribution change, motivating distribution-free sensitivity testing: "It’s a different approach to pathway analysis that defines pathway activity as a change in multivariate distribution of a given pathway across conditions"
- [abstract] SCPA is a distribution-free statistical framework for multisample distribution testing: "a sensitive and distribution-free statistical framework for multisample distribution testing"
