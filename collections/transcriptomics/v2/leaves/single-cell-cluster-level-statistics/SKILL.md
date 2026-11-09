---
name: single-cell-cluster-level-statistics
description: Use when CITE-seq data with normalized RNA and ADT matrices plus per-cell
  cluster assignments needs a quantitative per-cluster purity assessment in both transcriptome
  and surface protein modalities.
when_to_use_negative:
- Datasets with very large numbers of ADT features (e.g., 228 ADT features in 170K
  cells), where the paper shows LinQ-View fails and recommends Seurat 4
- Datasets beyond LinQ-View's sweet spot of large datasets with a limited number of
  ADT features (<50), since pairwise distance memory/complexity grow quadratically
  with cell count
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: LinQ-View
  role: Toolkit that defines the purity score; its ADT purity scoring uses the sum
    of standard deviations of ADT features as the diversity index
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: ROGUE
  role: Provides the entropy-based formula for the transcriptome purity score S_RNA
    per cluster
  repo: https://github.com/PaulingLiu/ROGUE
- name: R v3.6.3
  role: Runtime environment in which the purity scoring module is implemented and
    executed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:eed06766b31d83fab50375f86c0c4aa293117642d7b90ab57b45943b3a4cac83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - LinQ-View v0.99
  - R v3.6.3
  - ROGUE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# single-cell-cluster-level-statistics

## Summary

Computes per-cluster purity scores for CITE-seq data by combining an entropy-based ROGUE transcriptome score (S_RNA) with a rank-adjusted ADT score (S_ADT) via the geometric mean S = sqrt(S_ADT*S_RNA), providing a quantitative metric for judging whether a cluster is a pure cell population in both modalities.

## When to use

Use when CITE-seq data with normalized RNA and ADT matrices plus per-cell cluster assignments needs a quantitative per-cluster purity assessment in both transcriptome and surface protein modalities.

## When NOT to use

- Datasets with very large numbers of ADT features (e.g., 228 ADT features in 170K cells), where the paper shows LinQ-View fails and recommends Seurat 4
- Datasets beyond LinQ-View's sweet spot of large datasets with a limited number of ADT features (<50), since pairwise distance memory/complexity grow quadratically with cell count

## Inputs

- normalized RNA expression matrix
- normalized ADT (surface protein) expression matrix
- per-cell cluster assignments
- pre-defined technology-dependent parameter K for the ROGUE score

## Outputs

- purity_scores.csv with per-cluster S_RNA, S_ADT, and combined geometric-mean S values, all in [0,1]

## How to apply

Load the normalized RNA expression matrix, ADT expression matrix, and per-cell cluster assignments into R (v3.6.3). For each cluster, compute the entropy-based transcriptome score S_RNA using the ROGUE formula (PaulingLiu/ROGUE) with the pre-defined, technology-dependent parameter K. For each cluster, rank ADT features by variance and compute S_ADT = 1 - ΣSD_n/(ΣSD_n + K_ADT), summing standard deviations of the top n features with n = floor(ln(N)), where N is the number of ADT features; the sum of standard deviations of ADT features serves as the diversity index. Combine the modality scores into the final purity score with the geometric mean S = sqrt(S_ADT*S_RNA), then assemble per-cluster S_RNA, S_ADT, and S values into purity_scores.csv.

## Related tools

- **LinQ-View** (Toolkit that defines the purity score; its ADT purity scoring uses the sum of standard deviations of ADT features as the diversity index) — https://github.com/WilsonImmunologyLab/LinQView
- **ROGUE** (Provides the entropy-based formula for the transcriptome purity score S_RNA per cluster) — https://github.com/PaulingLiu/ROGUE
- **R v3.6.3** (Runtime environment in which the purity scoring module is implemented and executed) — https://www.r-project.org/

## Examples

```
devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Every reported S_RNA, S_ADT, and S value lies within [0,1]
- Each value reproduces the corresponding STAR Methods formula when recomputed independently
- S equals the geometric mean sqrt(S_ADT*S_RNA) for every cluster
- The number of top ADT features summed (n) equals floor(ln(N)) for the dataset's ADT feature count N
- Per-cluster S_RNA values match outputs of the ROGUE implementation (PaulingLiu/ROGUE) given the same K

## Limitations

- K is a pre-defined parameter dependent on single-cell sequencing technology, and its value is not specified in the article text
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features whereas Seurat 4 succeeded
- Memory usage and complexity of pairwise distance calculation grow quadratically as dataset size increases, limiting time efficiency

## Evidence

- [intro] we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality"
- [readme] Here, we proposed the concept of cluster purity and introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population.: [evidence span withheld — anchor status: not_found]
- [discussion] K is a pre-defined parameter and is dependent on single cell sequencing technology: "K is a pre-defined parameter and is dependent on single cell sequencing technology"
- [discussion] LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50): "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
