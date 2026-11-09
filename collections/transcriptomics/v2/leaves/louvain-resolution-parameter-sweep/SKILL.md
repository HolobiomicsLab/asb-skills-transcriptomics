---
name: louvain-resolution-parameter-sweep
description: Use when comparing multimodal single-cell clustering methods on the same
  CITE-seq dataset and you need a fair resolution-matched comparison plus a resolution
  sweep to explain divergent subset recovery and time efficiency.
when_to_use_negative:
- Massive datasets (e.g., 170K cells with 228 ADT features) where LinQ-View and other
  methods fail but Seurat 4 succeeds — pairwise distance computation scales quadratically
- Datasets with a large number of ADT features (>50), where LinQ-View is no longer
  the best option
- When the goal is unimodal scRNA-seq clustering rather than multimodal transcriptome-plus-surface-protein
  integration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: Multimodal integration method that fuses per-modality cell-cell pairwise distance
    matrices with an L-infinity norm, then clusters via Seurat-implemented Louvain
    community detection at resolution 0.9
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat 4
  role: Multimodal WNN integration baseline clustered with Louvain at resolution 0.9;
    fastest method (~20 s) in the comparison
  repo: https://satijalab.org/seurat/
- name: MOFA2
  role: Factor-analysis baseline clustered with Louvain at resolution 0.9 and re-run
    at resolution 1.5 for the parameter sweep
- name: SNF
  role: Similarity network fusion baseline clustered with Louvain at resolution 0.9;
    slowest method (3-4 h) in the comparison
- name: R v3.6.3
  role: Runtime environment for all methods and clustering comparisons
  repo: https://www.r-project.org/
- name: ggplot2
  role: Visualization of the Figure 6-style side-by-side subset comparison
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:f3cb249b35f7e56874a35449155beae66c7b4d2d3b609319ade58729626999ef
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# louvain-resolution-parameter-sweep

## Summary

Benchmarking protocol for multimodal CITE-seq clustering that compares methods (LinQ-View, Seurat 4, MOFA2, SNF) at a matched Louvain resolution (0.9) and sweeps resolution (e.g., MOFA2 at 1.5) to distinguish true method failure from parameter-driven differences in recovering NK and CD4+ T cell subsets, while recording runtime per method.

## When to use

Use when comparing multimodal single-cell clustering methods on the same CITE-seq dataset and you need a fair resolution-matched comparison plus a resolution sweep to explain divergent subset recovery and time efficiency.

## When NOT to use

- Massive datasets (e.g., 170K cells with 228 ADT features) where LinQ-View and other methods fail but Seurat 4 succeeds — pairwise distance computation scales quadratically
- Datasets with a large number of ADT features (>50), where LinQ-View is no longer the best option
- When the goal is unimodal scRNA-seq clustering rather than multimodal transcriptome-plus-surface-protein integration

## Inputs

- CITE-seq dataset GSE100866 (8,617 cord blood mononuclear cells) with transcriptome and surface-protein (ADT) modalities
- Pre-processed Seurat objects (post cell QC, normalization, scaling, HVG identification, PCA) per method
- Louvain resolution settings (0.9 matched; 1.5 for MOFA2 re-run)

## Outputs

- Figure 6-style side-by-side UMAP/cluster comparison of NK and CD4+ T cell subsets recovered by LinQ-View, Seurat 4, MOFA2, and SNF
- runtime_comparison.csv with elapsed runtime per method
- Per-method cluster counts and subset assignments at resolution 0.9 (and 1.5 for MOFA2)

## How to apply

Load the benchmark CITE-seq dataset (8,617 cord blood mononuclear cells, GSE100866) and apply the shared pre-processing chain: cell QC, normalization and scaling of both transcriptome and surface-protein modalities, HVG identification, and PCA. Run each method — LinQ-View (fusing per-modality pairwise distance matrices with an L-infinity norm), Seurat 4 multimodal integration, MOFA2, and SNF — followed by Seurat-implemented Louvain community detection clustering at the matched resolution 0.9. Because MOFA2 fails to separate the two NK cell subsets at 0.9, re-run it at a higher resolution (1.5) to test whether subset recovery is parameter-dependent rather than method-intrinsic. Annotate recovered NK and CD4+ T cell subsets per method (CD4+ T cells resolved as 2, 4, and 5 clusters by LinQ-View, MOFA2, and Seurat 4, respectively) and record elapsed runtimes (LinQ-View ~30 s, Seurat 4 ~20 s, MOFA2 ~6 min, SNF 3-4 h) into runtime_comparison.csv. The sweep's rationale is that comparing methods only at a single resolution can conflate clustering granularity with integration quality, so both the matched-resolution result and the swept-resolution result must be reported together.

## Related tools

- **LinQ-View** (Multimodal integration method that fuses per-modality cell-cell pairwise distance matrices with an L-infinity norm, then clusters via Seurat-implemented Louvain community detection at resolution 0.9) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat 4** (Multimodal WNN integration baseline clustered with Louvain at resolution 0.9; fastest method (~20 s) in the comparison) — https://satijalab.org/seurat/
- **MOFA2** (Factor-analysis baseline clustered with Louvain at resolution 0.9 and re-run at resolution 1.5 for the parameter sweep)
- **SNF** (Similarity network fusion baseline clustered with Louvain at resolution 0.9; slowest method (3-4 h) in the comparison)
- **R v3.6.3** (Runtime environment for all methods and clustering comparisons) — https://www.r-project.org/
- **ggplot2** (Visualization of the Figure 6-style side-by-side subset comparison) — https://ggplot2.tidyverse.org/

## Examples

```
Rscript run_method_comparison.R --dataset GSE100866 --methods LinQ-View,Seurat4,MOFA2,SNF --louvain.resolution 0.9 --mofa2.sweep.resolution 1.5 --runtime.out runtime_comparison.csv
```

## Evaluation signals

- LinQ-View and Seurat 4 recover two NK cell subsets at Louvain resolution 0.9 on GSE100866
- SNF and MOFA2 (at resolution 0.9) do NOT separate any NK cell subsets
- MOFA2 succeeds in resolving the NK cell subsets when re-run at resolution 1.5
- CD4+ T cell subsets are recovered as 2, 4, and 5 clusters by LinQ-View, MOFA2, and Seurat 4, respectively
- Measured runtimes reproduce the ordering LinQ-View/Seurat 4 (tens of seconds) << MOFA2 (~6 min) << SNF (3-4 h)

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency on large datasets
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded
- MOFA2's failure to separate NK subsets at resolution 0.9 is parameter-dependent, so single-resolution comparisons can mislead

## Evidence

- [results] On the benchmark dataset at the same resolution, LinQ-View and Seurat 4 correctly identified two NK cell subsets, while the other multimodal methods (MOFA2 and SNF) failed to separate any of these subsets.: [evidence span withheld — anchor status: not_found]
- [intro] L∞ norm fusion and Seurat Louvain clustering: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] Default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] Seurat 4 best for massive data: "Considering both performance and time efficiency, Seurat 4 is the best option for massive CITE-seq datasets with a high number of ADT features"
- [readme] README workflow description: "We reason that combining dissimilarity representations may help emphasize different information from each modality to maximally capture and infer cell heterogeneity"
