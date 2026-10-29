---
name: entropy-based-transcriptome-purity-scoring
description: Use when CITE-seq clusters must be assessed for transcriptomic homogeneity,
  i.e., per-cluster S_RNA values are needed (optionally to be combined with an ADT
  score via geometric mean) for cluster purity assessment.
when_to_use_negative:
- Datasets are massive with a very high number of ADT features — the article notes
  LinQ-View is best suited for large datasets with a limited number of ADT features
  (<50) and failed on 170K cells with 228 ADT features where Seurat 4 succeeded
- The K parameter for the ROGUE score cannot be set appropriately for the sequencing
  technology used, since its value is not specified in the article text
- Input clusters already come from a unimodal scRNA-seq-only workflow where ADT integration
  and modality-combined purity are not relevant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: Toolkit whose purity module wraps the ROGUE formula to compute S_RNA and combine
    it with the ADT score via geometric mean
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: ROGUE
  role: Provides the entropy-based ROGUE statistic used to compute the transcriptome
    purity score S_RNA per cluster
  repo: https://github.com/PaulingLiu/ROGUE
- name: R v3.6.3
  role: Runtime environment in which the purity scoring module was implemented and
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:1c862518e5b16bd73e400a6e39823ea1f631a4f6ad8b095089b9a1da8b60d29b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# entropy-based-transcriptome-purity-scoring

## Summary

Computes an entropy-based transcriptome purity score (S_RNA) per cell cluster using the ROGUE formula, as implemented in the LinQ-View CITE-seq purity module, to quantify whether a cluster is a uniform single-cell population.

## When to use

Use when CITE-seq clusters must be assessed for transcriptomic homogeneity, i.e., per-cluster S_RNA values are needed (optionally to be combined with an ADT score via geometric mean) for cluster purity assessment.

## When NOT to use

- Datasets are massive with a very high number of ADT features — the article notes LinQ-View is best suited for large datasets with a limited number of ADT features (<50) and failed on 170K cells with 228 ADT features where Seurat 4 succeeded
- The K parameter for the ROGUE score cannot be set appropriately for the sequencing technology used, since its value is not specified in the article text
- Input clusters already come from a unimodal scRNA-seq-only workflow where ADT integration and modality-combined purity are not relevant

## Inputs

- Normalized RNA expression matrix from CITE-seq data
- Per-cell cluster assignments (e.g., from Louvain community detection in Seurat)
- Pre-defined technology-dependent ROGUE parameter K
- (For the combined score) normalized ADT expression matrix

## Outputs

- Per-cluster S_RNA transcriptome purity scores in [0,1]
- (For the combined module) per-cluster S_ADT and geometric-mean S values assembled into purity_scores.csv

## How to apply

Load the normalized RNA expression matrix and per-cell cluster assignments into R (LinQ-View was implemented in R v3.6.3). For each cluster, compute the entropy-based transcriptome score S_RNA using the ROGUE formula from the PaulingLiu/ROGUE package, supplying the pre-defined technology-dependent parameter K (its value is not specified in the article text and must be chosen per sequencing technology). In LinQ-View's full module, S_RNA is subsequently combined with the rank-adjusted ADT score S_ADT = 1 - ΣSD_n/(ΣSD_n + K_ADT) — where ΣSD_n sums standard deviations of the top n = floor(ln(N)) ADT features ranked by variance — via the geometric mean S = sqrt(S_ADT*S_RNA). The rationale is that unimodal metrics are insufficient for CITE-seq, where surface protein variation can mask transcriptomic heterogeneity; ROGUE examines whether a given cluster is a pure cell population. Validate that every reported score lies within [0,1] and reproduces the corresponding STAR Methods formula before writing outputs such as purity_scores.csv.

## Related tools

- **LinQ-View** (Toolkit whose purity module wraps the ROGUE formula to compute S_RNA and combine it with the ADT score via geometric mean) — https://github.com/WilsonImmunologyLab/LinQView
- **ROGUE** (Provides the entropy-based ROGUE statistic used to compute the transcriptome purity score S_RNA per cluster) — https://github.com/PaulingLiu/ROGUE
- **R v3.6.3** (Runtime environment in which the purity scoring module was implemented and executed)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Every reported S_RNA (and combined S) value lies within [0,1]
- Each computed value reproduces the corresponding STAR Methods formula when recalculated independently
- Per-cluster S_RNA, S_ADT, and S values assemble into a complete table written as purity_scores.csv with one row per cluster
- ROGUE installation and dependency check passes: the tidyverse package is installed before devtools::install_github("PaulingLiu/ROGUE")
- Pure (uniform) clusters yield high purity scores while demonstrably heterogeneous clusters yield lower scores, consistent with the ROGUE concept of cluster purity

## Limitations

- The pre-defined parameter K for the ROGUE score is technology-dependent but its value is not specified in the article text, requiring user calibration per sequencing technology
- LinQ-View's pairwise distance computation scales quadratically, limiting time efficiency with increasing cell numbers
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded
- ROGUE was originally framed for unsupervised scRNA-seq clusters; uniformity judgments depend on the clustering quality fed into the score

## Evidence

- [intro] we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data"
- [readme] ROGUE concept: [evidence span withheld — anchor status: not_found]
- [discussion] K parameter caveat: "K is a pre-defined parameter and is dependent on single cell sequencing technology"
- [methods] ROGUE workflow step: [evidence span withheld — anchor status: not_found]
