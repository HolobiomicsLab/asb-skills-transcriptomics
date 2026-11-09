---
name: cd4-t-cell-naive-memory-discrimination
description: Use when analyzing CITE-seq data (e.g., GSE100866 CBMCs) where naive
  vs. memory CD4+ T cells (or CD8−/CD8+ NK cells) co-cluster under RNA-only or ADT-only
  clustering and joint multimodal clustering is needed to resolve them.
when_to_use_negative:
- Massive CITE-seq datasets with many ADT features (e.g., 170K cells with 228 ADT
  features) — LinQ-View failed on such data whereas Seurat 4 succeeded
- Large datasets with a high number of ADT features (>50) — LinQ-View is best suited
  for datasets with a limited number of ADT features (e.g., <50); Seurat 4 is the
  better option for massive CITE-seq datasets
- Purely unimodal scRNA-seq data with no surface-protein modality, since the modality-exclusive
  resolution depends on fusing RNA and ADT distance matrices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0121
tools:
- name: LinQ-View
  role: 'Performs the joint multimodal clustering: fuses per-modality pairwise distance
    matrices with the L∞ norm model, applies the sigmoid filter to ADT distances,
    and computes quantitative purity scores'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Implements the default Louvain community detection clustering used for both
    unimodal (RNA-only, ADT-only) and LinQ-View joint clustering
  repo: https://satijalab.org/seurat/
- name: ROGUE
  role: Underlying cluster-purity statistic (Ratio of Global Unshifted Entropy) used
    to assess whether clusters are pure cell populations in the CITE-seq data
  repo: https://github.com/PaulingLiu/ROGUE
- name: dplyr
  role: Data manipulation when writing purity scores and cluster assignments to figure4_purity_scores.csv
  repo: https://dplyr.tidyverse.org/
- name: ggplot2
  role: Visualization of clusters and purity scores matching Figure 4
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:fcce6d9249a8a272de27c84ae0cef89f048416eb40d9c8fcf7ba23b3c0da4d73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - LinQ-View v0.99
  - R
  - ROGUE
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cd4-t-cell-naive-memory-discrimination

## Summary

A LinQ-View-based workflow that resolves modality-exclusive CD4+ T cell subsets (naive vs. memory CD4+ T cells) in CITE-seq data by fusing transcriptome and surface-protein distance matrices with an L∞ norm, demonstrating that unimodal RNA- or ADT-only Louvain clustering cannot separate them.

## When to use

Use when analyzing CITE-seq data (e.g., GSE100866 CBMCs) where naive vs. memory CD4+ T cells (or CD8−/CD8+ NK cells) co-cluster under RNA-only or ADT-only clustering and joint multimodal clustering is needed to resolve them.

## When NOT to use

- Massive CITE-seq datasets with many ADT features (e.g., 170K cells with 228 ADT features) — LinQ-View failed on such data whereas Seurat 4 succeeded
- Large datasets with a high number of ADT features (>50) — LinQ-View is best suited for datasets with a limited number of ADT features (e.g., <50); Seurat 4 is the better option for massive CITE-seq datasets
- Purely unimodal scRNA-seq data with no surface-protein modality, since the modality-exclusive resolution depends on fusing RNA and ADT distance matrices

## Inputs

- Processed CITE-seq Seurat object with transcriptome and surface-protein (ADT) modalities (Dataset 1: GSE100866, 8,617 CBMCs)
- Cell-cell pairwise distance matrices per modality (RNA and ADT)
- Cluster assignments from RNA-only, ADT-only, and joint LinQ-View clustering

## Outputs

- Cluster assignments for all three clusterings (RNA-only, ADT-only, LinQ-View joint) showing resolution of naive and memory CD4+ T cell subsets
- Purity scores per cluster in both transcriptome and surface-protein modalities for each clustering method
- figure4_purity_scores.csv with purity scores and cluster assignments

## How to apply

Load the processed CITE-seq object (Dataset 1, GSE100866, 8,617 cord blood mononuclear cells) with both modalities in R, then run three clusterings: RNA-only Louvain, ADT-only Louvain (with LinQ-View's sigmoid filter on the ADT distance calculation), and LinQ-View joint clustering, which fuses per-modality cell-cell pairwise distance matrices using the L∞ norm model. The default clustering method is the Louvain community detection algorithm implemented in Seurat. Compute LinQ-View's quantitative purity score for every cluster in both the transcriptome and surface-protein modalities for each clustering. Confirm the naive and memory CD4+ T cell subsets (alongside CD8− and CD8+ NK cells) are resolved only by the joint clustering, and write purity scores and cluster assignments for all three methods to figure4_purity_scores.csv.

## Related tools

- **LinQ-View** (Performs the joint multimodal clustering: fuses per-modality pairwise distance matrices with the L∞ norm model, applies the sigmoid filter to ADT distances, and computes quantitative purity scores) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Implements the default Louvain community detection clustering used for both unimodal (RNA-only, ADT-only) and LinQ-View joint clustering) — https://satijalab.org/seurat/
- **ROGUE** (Underlying cluster-purity statistic (Ratio of Global Unshifted Entropy) used to assess whether clusters are pure cell populations in the CITE-seq data) — https://github.com/PaulingLiu/ROGUE
- **dplyr** (Data manipulation when writing purity scores and cluster assignments to figure4_purity_scores.csv) — https://dplyr.tidyverse.org/
- **ggplot2** (Visualization of clusters and purity scores matching Figure 4) — https://ggplot2.tidyverse.org/

## Examples

```
devtools::install_github("PaulingLiu/ROGUE")  # purity-score package used alongside LinQ-View (https://WilsonImmunologyLab.github.io/LinQView/)
```

## Evaluation signals

- The naive and memory CD4+ T cell subsets (and CD8−/CD8+ NK cells) are resolved only by the LinQ-View joint clustering, not by RNA-only or ADT-only Louvain clustering
- Purity scores are computed for every cluster in both the transcriptome and surface-protein modalities for all three clusterings; the NK cell clusters achieve RNA purity of 0.9553 and 0.9434 and ADT purity of 0.7822 and 0.8279
- Joint distance matrix is produced by fusing per-modality cell-cell pairwise distance matrices with the L∞ norm model, and the ADT distances pass through LinQ-View's sigmoid filter
- Output file figure4_purity_scores.csv contains purity scores and cluster assignments for all three methods
- Cell set matches the expected 8,617 CBMCs from GSE100866 after QC (cells with <200 or >2500 detected genes excluded)

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time and memory efficiency as dataset size increases
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded
- LinQ-View is best suited to large datasets with a limited number of ADT features (e.g., <50)

## Evidence

- [intro] L∞ norm fusion of modalities: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [intro] purity score metric: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data"
- [discussion] default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] sigmoid filter on ADT: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [readme] LinQ-View purpose: "Integrate multiple modalities of CITE-seq data for single cell populations identification"
