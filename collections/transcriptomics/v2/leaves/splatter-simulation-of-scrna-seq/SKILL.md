---
name: splatter-simulation-of-scrna-seq
description: Use when a clustering benchmark requires a large simulated scRNA-seq
  dataset with known ground-truth labels — e.g. stress-testing a method's scalability
  to ~1,000,000 cells when real annotated data of that size is unavailable.
when_to_use_negative:
- The benchmark target is real experimental data with expert-assigned ground truth
  (e.g. Levine_32dim mass cytometry from FlowRepository FR-FCM-ZZPH) rather than simulated
  transcriptomes.
- The simulation targets non-transcriptomic data types (e.g. mass cytometry marker
  panels), which Splatter's scRNA-seq count model does not represent.
- Small generic clustering toy benchmarks are needed — the paper used scikit-learn
  random sample generators (1,000 objects, seed=1) for those, not Splatter.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: 'Primary simulator: estimates parameters from a reference scRNA-seq dataset
    and generates the million-cell benchmark with known labels'
- name: scikit-learn
  role: Alternative simulator used in the paper for small generic simulated clustering
    datasets (1,000 objects, seed=1)
- name: KMDHierarchicalClustering
  role: 'Downstream consumer: the Splatter-simulated million-cell dataset is used
    to test sampled KMD clustering scalability'
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: glmpca
  role: Source of the data simulation procedure used for the Townes et al. simulated
    scRNA-seq dataset
  repo: https://github.com/willtownes/glmpca/blob/master/vignettes/glmpca.Rmd
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:8ef17a25945271df3467d387146b56277f176ccb235cdf7a10373d5729096ead
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - MATLAB
  - Python
  - SCCAF
  - Scanpy
  - Splatter
  - glmpca
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- scrna-seq-simulation-with-splatter
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# splatter-simulation-of-scrna-seq

## Summary

Generate large simulated single-cell RNA-seq datasets with Splatter, estimating simulation parameters from a real reference dataset (e.g. Zeisel15) and fixing a seed, to create million-scale benchmark data with known ground-truth cell labels for evaluating clustering scalability.

## When to use

Use when a clustering benchmark requires a large simulated scRNA-seq dataset with known ground-truth labels — e.g. stress-testing a method's scalability to ~1,000,000 cells when real annotated data of that size is unavailable.

## When NOT to use

- The benchmark target is real experimental data with expert-assigned ground truth (e.g. Levine_32dim mass cytometry from FlowRepository FR-FCM-ZZPH) rather than simulated transcriptomes.
- The simulation targets non-transcriptomic data types (e.g. mass cytometry marker panels), which Splatter's scRNA-seq count model does not represent.
- Small generic clustering toy benchmarks are needed — the paper used scikit-learn random sample generators (1,000 objects, seed=1) for those, not Splatter.

## Inputs

- Reference scRNA-seq dataset for parameter estimation (e.g. Zeisel15: 3,005 mouse cerebral cortex cells)
- Simulation settings: number of cells (e.g. 1,000,000), random seed (e.g. 1), expected number of cell populations

## Outputs

- Simulated scRNA-seq count matrix (e.g. 1,000,000 cells x genes) from Splatter
- Ground-truth per-cell population labels (y_true) for scoring clusterings with ARI/NMI/accuracy
- Reproducible benchmark artifact (fixed seed) reusable across compared methods

## How to apply

Estimate Splatter's simulation parameters from a real reference scRNA-seq dataset; in the KMD clustering work the reference was Zeisel15 (3,005 mouse cerebral cortex cells). Simulate with the target cell count (1,000,000 cells for the million-object scaling test) and a fixed seed (seed=1) so the run is reproducible. Retain the simulated true cell-type labels as y_true for computing ARI, NMI and accuracy against clusterings produced on the simulated data. Note that for smaller general-purpose clustering benchmarks the same paper instead generated simulated data with scikit-learn random sample generators (1,000 objects, seed=1), and the Townes et al. scRNA-seq simulation followed the glmpca vignette procedure — choose Splatter specifically when you need transcriptomic count-like data with tunable scale and built-in group structure.

## Related tools

- **Splatter** (Primary simulator: estimates parameters from a reference scRNA-seq dataset and generates the million-cell benchmark with known labels)
- **scikit-learn** (Alternative simulator used in the paper for small generic simulated clustering datasets (1,000 objects, seed=1))
- **KMDHierarchicalClustering** (Downstream consumer: the Splatter-simulated million-cell dataset is used to test sampled KMD clustering scalability) — github.com/KaplanLab/KMDHierarchicalClustering
- **glmpca** (Source of the data simulation procedure used for the Townes et al. simulated scRNA-seq dataset) — https://github.com/willtownes/glmpca/blob/master/vignettes/glmpca.Rmd

## Examples

```
library(Splatter); params <- splatEstimate(zeisel15_counts); sim <- splatSimulate(params, nCells = 1000000, seed = 1)
```

## Evaluation signals

- The simulated dataset contains ground-truth labels, so a downstream clustering can be scored with ARI/NMI/accuracy; on the Splatter million-cell dataset, sampled KMD at a 5,000-cell core was expected to reach ~0.956 accuracy.
- Re-running the simulation with the same parameters and seed=1 reproduces an identical count matrix (fixed-seed reproducibility check).
- The simulated cell count matches the requested nCells (e.g. 1,000,000 cells) and the number of simulated groups corresponds to the parameterized populations.
- Parameter estimation converges on the reference dataset (Zeisel15, 3,005 cells) without error before simulation begins.

## Limitations

- Simulation quality is bounded by the parameter-estimation dataset (Zeisel15 has only 3,005 cells), so very small or rare populations may not be represented in the reference parameters.
- When the simulated data is later analyzed with a sampling-based method, very small clusters may be missed if they are not sampled well.
- The article reports no software version numbers for the tools used, so exact Splatter version provenance is not documented.

## Evidence

- [other] Splatter million-cell simulation workflow: [evidence span withheld — anchor status: not_found]
- [discussion] scikit-learn generators for small sims: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [discussion] Zeisel15 reference dataset: "The Zeisel15 dataset consists single cell transcriptome of 3005 mice cerebral cortex cells."
- [discussion] Townes et al. simulation procedure: "The Townes et al.30 simulated dataset was created according to the data simulation procedure"
