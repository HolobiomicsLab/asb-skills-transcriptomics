---
name: controlled-single-cell-simulation-generation
description: Use when a multi-omics integration method must be benchmarked against
  a known ground truth, e.g. simulating mixed or rare populations and controlled sparsity
  (82/90/96%) in paired scRNA/scATAC cell-line data.
when_to_use_negative:
- Datasets lack distinct, known subpopulations to mix or subsample — the simulation
  needs a clear ground truth
- The evaluation goal is biological discovery rather than controlled method benchmarking;
  artificial perturbations would distort real structure
- Unpaired single-modality data, since the settings exploit jointly profiled RNA and
  ATAC from the same cells
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Scanpy
  role: Loads and preprocesses the Liu scCAT-seq data before controlled settings are
    constructed
- name: Muon
  role: Handles the paired RNA/ATAC MuData object during construction of the controlled
    settings
- name: Mowgli
  role: Method evaluated on the simulated controlled settings to verify the benchmark's
    discriminative power
  repo: github.com/cantinilab/mowgli
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:a78ba1b5c1146e9c641e0a7dc460aedb04899a61c6f52cb02b3c4d513ff4bc95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Muon
  - Python
  - Scanpy
  - Seurat v4
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# controlled-single-cell-simulation-generation

## Summary

Constructs controlled benchmark settings from paired single-cell multi-omics data by surgically mixing profiles across cell types and randomly zeroing values to simulate confounding and sparsity, enabling ground-truth evaluation of integration methods on the Liu scCAT-seq RNA/ATAC cell-line dataset (HCT116, HeLa-S3, K562).

## When to use

Use when a multi-omics integration method must be benchmarked against a known ground truth, e.g. simulating mixed or rare populations and controlled sparsity (82/90/96%) in paired scRNA/scATAC cell-line data.

## When NOT to use

- Datasets lack distinct, known subpopulations to mix or subsample — the simulation needs a clear ground truth
- The evaluation goal is biological discovery rather than controlled method benchmarking; artificial perturbations would distort real structure
- Unpaired single-modality data, since the settings exploit jointly profiled RNA and ATAC from the same cells

## Inputs

- Preprocessed Liu scCAT-seq paired RNA/ATAC matrix (HCT116, HeLa-S3, K562 cell lines)
- Cell-type labels for each cell line
- Random seed / sampling specification for selecting HeLa cells and zeroing entries

## Outputs

- Controlled benchmark settings: baseline, Mixed in RNA, Mixed in both, Rare population
- Sparsity-perturbed variants at 82%, 90%, and 96% final sparsity
- Preserved ground-truth cluster labels per setting for clustering evaluation

## How to apply

Start from the preprocessed Liu scCAT-seq dataset with joint RNA/ATAC profiling of HCT116, HeLa-S3, and K562 cell lines. Build the controlled settings by targeted cell manipulation: replace HCT116 RNA profiles with random HeLa RNA profiles for 'Mixed in RNA'; additionally replace K562 ATAC with HeLa ATAC for 'Mixed in both'; retain only 10 randomly sampled HeLa cells to create a 'Rare population'. Then simulate sparsity by randomly zeroing 50%, 70%, or 90% of data values to reach 82%, 90%, or 96% final sparsity. These manipulations preserve the known three-population ground truth so clustering performance (silhouette score, maximum ARI, kNN purity) can be attributed to each method's robustness rather than annotation uncertainty.

## Related tools

- **Scanpy** (Loads and preprocesses the Liu scCAT-seq data before controlled settings are constructed)
- **Muon** (Handles the paired RNA/ATAC MuData object during construction of the controlled settings)
- **Mowgli** (Method evaluated on the simulated controlled settings to verify the benchmark's discriminative power) — github.com/cantinilab/mowgli

## Examples

```
git clone https://github.com/cantinilab/mowgli_reproducibility && python scripts/liu_simulation.py  # construct Mixed in RNA, Mixed in both, Rare population, and 50/70/90%-zeroed settings
```

## Evaluation signals

- Final sparsity levels match targets: 82%, 90%, and 96% after zeroing 50%, 70%, and 90% of values respectively
- Rare population setting contains exactly 10 randomly sampled HeLa cells alongside the other populations
- Mixed settings retain the three-cell-line ground-truth labels so ARI, silhouette, and kNN purity are computable
- All six benchmark methods (Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, Multigrate) run on identical perturbed matrices
- Expected sanity check: methods confuse the three populations in 96% sparse while Mowgli separates them, mirroring Fig. 2

## Limitations

- Simulation is limited to cell-line data with clear a priori populations; real tissues with ambiguous annotation cannot provide this ground truth
- Controlled sparsity is simulated by random zeroing, which may not faithfully model dropout mechanisms in real sparse single-cell data
- Only RNA and ATAC modalities are simulated here; the scheme would need extension for ADT or additional omics

## Evidence

- [results] Controlled settings derived from cell line data with known ground truth: [evidence span withheld — anchor status: not_found]
- [results] Performance comparison on controlled settings: "Indeed, in all datasets except 90% sparse, Mowgli has a performance greater or equal to that of other methods."
- [results] Methods fail at high simulated sparsity: [evidence span withheld — anchor status: not_found]
- [supplementary] Reproducibility code for controlled experiments: [evidence span withheld — anchor status: not_found]
