---
name: sensitivity-analysis-across-simulation-scenarios
description: Use when you must quantify and compare detection sensitivity of pathway
  analysis tools across a controlled panel of simulated two-population scRNA-seq comparisons
  with an implanted target pathway.
when_to_use_negative:
- The comparison uses real experimental populations rather than simulated data with
  a known ground-truth implanted pathway — sensitivity cannot be computed without
  the implanted target
- The goal is differential pathway ranking or biological interpretation of real conditions
  (e.g. COVID-19 vs healthy monocytes), where compare_pathways() results on real data
  are the appropriate output instead of a simulation sweep
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCPA
  role: Method under test; compare_pathways() is run on each simulated sample pair
    to detect the implanted pathway
  repo: https://github.com/jackbibby1/SCPA
- name: Splatter
  role: splatEstimate() estimates simulation parameters from the reference dataset;
    splatSimulate() generates the two simulated populations with implanted perturbation
- name: fGSEA
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: GSVA
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: AUCell
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: Vision
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: ssGSEA
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: iDEA
  role: Competitor method run on identical simulated datasets for sensitivity comparison
- name: Seurat
  role: Provides the reference single-cell data objects from which simulation parameters
    are estimated
- name: msigdbr
  role: Source of pathway gene sets used with compare_pathways()
- name: dplyr
  role: Data manipulation when assembling the sensitivity results table
- name: ggplot2
  role: Visualization of method-by-scenario sensitivity results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:ea434b66358773432d3318c16937e9d5fcab2f8230689ed9cbd974f59bafbd4a
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# sensitivity-analysis-across-simulation-scenarios

## Summary

Benchmark a pathway analysis method's sensitivity against competitors by implanting a perturbed 200-gene pathway into Splatter-simulated single-cell populations and recording the fraction of scenarios in which each method detects the implanted pathway as significant.

## When to use

Use when you must quantify and compare detection sensitivity of pathway analysis tools across a controlled panel of simulated two-population scRNA-seq comparisons with an implanted target pathway.

## When NOT to use

- The comparison uses real experimental populations rather than simulated data with a known ground-truth implanted pathway — sensitivity cannot be computed without the implanted target
- The goal is differential pathway ranking or biological interpretation of real conditions (e.g. COVID-19 vs healthy monocytes), where compare_pathways() results on real data are the appropriate output instead of a simulation sweep

## Inputs

- Reference single-cell dataset (e.g. Seurat object or expression matrix) for splatEstimate() parameter estimation
- 200-gene target pathway gene set matrix to implant the perturbation
- Panel of Splatter-simulated two-population sample pairs across varied perturbation scenarios

## Outputs

- sensitivity_benchmark.csv with one sensitivity entry per method per simulated scenario
- Method-by-method sensitivity values (fraction of scenarios where the implanted pathway is significant)

## How to apply

Estimate simulation parameters from a reference single-cell dataset with splatEstimate(), then generate simulated two-population datasets with splatSimulate(), using ~17,000 background genes plus a 200-gene pathway matrix carrying the implanted perturbation. Sweep simulation scenarios by varying perturbation strength/parameters to build a panel of simulated comparisons. Run SCPA's compare_pathways() on each simulated sample pair and run the competitors (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA) on the identical datasets. For each method, compute sensitivity as the fraction of scenarios in which the implanted 200-gene pathway is detected as significant, and write the method-by-scenario results to sensitivity_benchmark.csv. The rationale, per the Cell Reports paper, is that SCPA defines pathway activity as a change in multivariate distribution rather than enrichment, which the benchmark shows yields greater sensitivity than commonly used pathway analysis tools.

## Related tools

- **SCPA** (Method under test; compare_pathways() is run on each simulated sample pair to detect the implanted pathway) — https://github.com/jackbibby1/SCPA
- **Splatter** (splatEstimate() estimates simulation parameters from the reference dataset; splatSimulate() generates the two simulated populations with implanted perturbation)
- **fGSEA** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **GSVA** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **AUCell** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **Vision** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **ssGSEA** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **iDEA** (Competitor method run on identical simulated datasets for sensitivity comparison)
- **Seurat** (Provides the reference single-cell data objects from which simulation parameters are estimated)
- **msigdbr** (Source of pathway gene sets used with compare_pathways())
- **dplyr** (Data manipulation when assembling the sensitivity results table)
- **ggplot2** (Visualization of method-by-scenario sensitivity results)

## Examples

```
params <- splatEstimate(ref_sce); sims <- splatSimulate(params, nGenes = 17000); scpa_res <- compare_pathways(samples, pathways)
```

## Evaluation signals

- sensitivity_benchmark.csv contains exactly one sensitivity entry per method per simulated scenario (no missing method-scenario combinations)
- The same simulated datasets are fed to SCPA and all competitors (fGSEA, iDEA, GSVA, AUCell, Vision, ssGSEA), so differences reflect method sensitivity, not data differences
- Sensitivity values fall in [0, 1] and represent the fraction of scenarios in which the implanted 200-gene pathway is called significant
- SCPA shows higher sensitivity than the competitor methods across the swept scenarios, replicating the paper's finding
- Simulated datasets contain ~17,000 background genes plus the 200-gene pathway matrix carrying the implanted perturbation

## Limitations

- Results are specific to Splatter's simulation model; sensitivity conclusions depend on the parameter ranges and perturbation-strength scenarios swept
- Only sensitivity of detecting the implanted pathway is measured — false positive rates against the ~17,000 background genes are not the primary metric here
- SCPA's advantage is demonstrated in simulations; interpretation on real data should use qval as the primary statistic with fold change/enrichment as secondary

## Evidence

- [results] SCPA detected perturbed pathways with greater sensitivity than the commonly used pathway analysis tools: [evidence span withheld — anchor status: not_found]
- [methods] splatEstimate and splatSimulate workflow: [evidence span withheld — anchor status: not_found]
