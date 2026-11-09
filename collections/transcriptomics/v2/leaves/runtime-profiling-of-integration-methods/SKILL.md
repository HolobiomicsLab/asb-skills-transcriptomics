---
name: runtime-profiling-of-integration-methods
description: 'Decision support for choosing a multimodal integration method when run
  time matters: datasets such as CITE-seq CBMCs (GSE100866) processed with identical
  pre-processing, clustered at matched Louvain resolution, and timed per method.'
when_to_use_negative:
- 'Dataset is massive with many ADT features (e.g., 170K cells with 228 ADT features):
  LinQ-View and SNF fail or become impractical, so a runtime comparison including
  them is not informative — use Seurat 4 directly'
- 'The goal is cluster accuracy alone with no runtime constraint: profiling time adds
  no value if a single method is already predetermined'
- Datasets lack a matched surface-protein modality, since the integration methods
  being profiled (LinQ-View, Seurat 4 multimodal, MOFA2, SNF) require both modalities
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LinQ-View
  role: Fast multimodal integration method being timed; fuses pairwise distance matrices
    with an L-infinity norm and clusters via Seurat-implemented Louvain at resolution
    0.9
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat 4
  role: Fastest comparator (~20 s); recommended by the article for massive CITE-seq
    datasets with a high number of ADT features
- name: MOFA2
  role: Intermediate-speed comparator (~6 min) whose clustering resolution must be
    raised to 1.5 to resolve NK cell subsets
- name: SNF
  role: Slowest comparator (3-4 h) that fails to separate NK cell subsets at resolution
    0.9
- name: ggplot2
  role: Used to build the Figure 6-style side-by-side subset-recovery and runtime
    comparison plots
  repo: https://ggplot2.tidyverse.org/
- name: R v3.6.3
  role: Execution environment in which all four methods are run and timed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:edb50b676676184f99c8768e49a9b2a923374d250e9335e7e2566e7ab64ea254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - MOFA2
  - R v3.6.3
  - SNF
  - Seurat 4
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# runtime-profiling-of-integration-methods

## Summary

Benchmark the wall-clock runtime of multimodal CITE-seq integration methods (LinQ-View, Seurat 4, MOFA2, SNF) on the same pre-processed dataset while verifying they still recover known cell heterogeneity, to expose the performance-versus-speed trade-off.

## When to use

Decision support for choosing a multimodal integration method when run time matters: datasets such as CITE-seq CBMCs (GSE100866) processed with identical pre-processing, clustered at matched Louvain resolution, and timed per method.

## When NOT to use

- Dataset is massive with many ADT features (e.g., 170K cells with 228 ADT features): LinQ-View and SNF fail or become impractical, so a runtime comparison including them is not informative — use Seurat 4 directly
- The goal is cluster accuracy alone with no runtime constraint: profiling time adds no value if a single method is already predetermined
- Datasets lack a matched surface-protein modality, since the integration methods being profiled (LinQ-View, Seurat 4 multimodal, MOFA2, SNF) require both modalities

## Inputs

- CITE-seq dataset with matched transcriptome and surface-protein modalities (e.g., GSE100866, 8,617 cord blood mononuclear cells)
- Standard pre-processed Seurat object (post QC, normalization, scaling, HVG, PCA) shared across methods
- Louvain clustering resolution setting (0.9 for all methods; 1.5 re-run for MOFA2)

## Outputs

- runtime_comparison.csv with elapsed runtime per method (LinQ-View ~30 s, Seurat 4 ~20 s, MOFA2 ~6 min, SNF 3-4 h)
- Figure 6-style side-by-side comparison of recovered NK and CD4+ T cell subsets per method at matched resolution

## How to apply

Load Dataset 1 (8,617 cord blood mononuclear cells, GSE100866) and apply the shared pre-processing chain once: cell QC, normalization and scaling of both transcriptome and ADT modalities, HVG identification, and PCA. Run each integration method with Louvain clustering at the same resolution (0.9): LinQ-View (which fuses per-modality cell-cell pairwise distance matrices with an L-infinity norm), Seurat 4 multimodal integration, MOFA2 (also re-run at resolution 1.5), and SNF. Wrap each run in a timer (e.g., R system.time) and record elapsed runtimes to runtime_comparison.csv. Judge correctness both on runtime ordering — LinQ-View/Seurat 4 (tens of seconds) << MOFA2 (minutes) << SNF (hours) — and on biological validity, since LinQ-View and Seurat 4 recover two NK cell subsets at resolution 0.9 while SNF and MOFA2 (0.9) do not and MOFA2 succeeds only at resolution 1.5.

## Related tools

- **LinQ-View** (Fast multimodal integration method being timed; fuses pairwise distance matrices with an L-infinity norm and clusters via Seurat-implemented Louvain at resolution 0.9) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat 4** (Fastest comparator (~20 s); recommended by the article for massive CITE-seq datasets with a high number of ADT features)
- **MOFA2** (Intermediate-speed comparator (~6 min) whose clustering resolution must be raised to 1.5 to resolve NK cell subsets)
- **SNF** (Slowest comparator (3-4 h) that fails to separate NK cell subsets at resolution 0.9)
- **ggplot2** (Used to build the Figure 6-style side-by-side subset-recovery and runtime comparison plots) — https://ggplot2.tidyverse.org/
- **R v3.6.3** (Execution environment in which all four methods are run and timed) — https://www.r-project.org/

## Evaluation signals

- Runtime ordering reproduced: LinQ-View and Seurat 4 finish in tens of seconds, MOFA2 in minutes (~6 min), SNF in hours (3-4 h)
- At Louvain resolution 0.9, LinQ-View and Seurat 4 each recover two NK cell subsets while SNF and MOFA2 recover none
- MOFA2 resolves the two NK cell subsets only when re-run at resolution 1.5
- CD4+ T cell subsets recovered as 2 clusters (LinQ-View), 5 clusters (MOFA2), and 4 clusters (Seurat 4) at matched resolution
- All methods run on the identical pre-processed input object and identical clustering resolution, so runtime differences are attributable to the integration method alone

## Limitations

- LinQ-View's pairwise distance computation has quadratic memory/complexity growth, limiting its time efficiency as dataset size increases
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded
- Runtime is dataset- and hardware-dependent; the reported ordering (tens of seconds << minutes << hours) is established on the 8,617-cell CBMC benchmark and may not transfer
- Runtime comparisons are only meaningful when pre-processing and clustering resolution are held fixed across methods

## Evidence

- [discussion] Considering both performance and time efficiency, Seurat 4 is the best option for massive CITE-seq datasets with a high number of ADT features: "Considering both performance and time efficiency, Seurat 4 is the best option for massive CITE-seq datasets with a high number of ADT features"
- [discussion] Quadratic complexity of pairwise distances limits LinQ-View time efficiency: "the memory usage and complexity of pairwise distance calculation grow quadratically as the size of datasets increases, the current design of LinQ-View has a limitation in time efficiency"
- [discussion] Benchmark dataset is 8,617 CBMCs with GEO accession GSE100866: "a dataset of 8,617 cord blood mononuclear cells (CBMCs), produced with CITE-seq (Stoeckius et al., 2017). NCBI GEO accession number is GSE100866"
- [intro] LinQ-View fuses modality distances with an L-infinity norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] LinQ-View default clustering uses Seurat's community detection algorithm: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
