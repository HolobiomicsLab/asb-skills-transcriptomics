---
name: cluster-purity-score-computation
description: Use when CITE-seq clusters have been produced (e.g., by LinQ-View joint,
  RNA-only, or ADT-only Louvain clustering) and you need a quantitative purity value
  per cluster in each modality to compare clustering quality.
when_to_use_negative:
- The dataset is massive with a high number of ADT features — LinQ-View failed on
  170K cells with 228 ADT features where Seurat 4 succeeded; LinQ-View is best for
  large datasets with <50 ADT features
- The input is single-modality data (scRNA-seq only or flow-like ADT data only) —
  the purity score is designed for dual transcriptome/surface-protein CITE-seq modalities
- You need a definitive answer about whether LinQ-View resolved specific modality-exclusive
  subsets without also running the unimodal RNA and ADT clusterings as comparators
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View v0.99
  role: Provides the joint clustering (L∞ norm fusion of per-modality pairwise distances)
    and the quantitative purity-score metric applied per cluster per modality
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: ROGUE
  role: Implements the cluster-purity statistic (Ratio of Global Unshifted Entropy)
    underlying the purity score; installed as an R dependency from GitHub
  repo: https://github.com/PaulingLiu/ROGUE
- name: Seurat
  role: Runs the default Louvain community-detection clustering on RNA-only and ADT-only
    modalities that the purity scores are computed over
  repo: https://satijalab.org/seurat/
- name: dplyr
  role: Data wrangling when writing purity scores and cluster assignments to the output
    CSV
  repo: https://dplyr.tidyverse.org/
- name: ggplot2
  role: Visualization of purity scores per cluster and per modality
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:9b4c2d9442fd2b1a5b8710b4a682a2022009a3d6bfcff609f8b34c00306bbf19
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cluster-purity-score-computation

## Summary

Compute LinQ-View's quantitative purity score for every cluster in both the transcriptome and surface-protein (ADT) modalities of CITE-seq data, based on the ROGUE concept of examining whether a cluster is a pure cell population. This metric enables objective assessment of cluster homogeneity that unimodal analyses lack.

## When to use

Use when CITE-seq clusters have been produced (e.g., by LinQ-View joint, RNA-only, or ADT-only Louvain clustering) and you need a quantitative purity value per cluster in each modality to compare clustering quality.

## When NOT to use

- The dataset is massive with a high number of ADT features — LinQ-View failed on 170K cells with 228 ADT features where Seurat 4 succeeded; LinQ-View is best for large datasets with <50 ADT features
- The input is single-modality data (scRNA-seq only or flow-like ADT data only) — the purity score is designed for dual transcriptome/surface-protein CITE-seq modalities
- You need a definitive answer about whether LinQ-View resolved specific modality-exclusive subsets without also running the unimodal RNA and ADT clusterings as comparators

## Inputs

- Processed CITE-seq Seurat object with transcriptome and surface-protein (ADT) modalities (e.g., GSE100866, 8,617 CBMCs)
- Cluster assignments from RNA-only, ADT-only, and LinQ-View joint Louvain clustering
- Per-modality cell-cell pairwise distance matrices (ADT distances passed through LinQ-View's sigmoid filter)

## Outputs

- Purity scores for every cluster in the transcriptome modality and surface-protein modality for each clustering method
- CSV of purity scores and cluster assignments for all three methods (figure4_purity_scores.csv)

## How to apply

After clustering (default: Louvain community detection as implemented in Seurat, with the L∞ norm joint distance from LinQ-View), apply LinQ-View's purity metric — implemented via the ROGUE statistic (Ratio of Global Unshifted Entropy) — separately to the transcriptome and surface-protein modalities. For Dataset 1 (GSE100866, 8,617 CBMCs), compute the purity score for every cluster from all three clusterings (RNA-only, ADT-only with the sigmoid-filtered ADT distance, and LinQ-View joint). As a validation benchmark, the two LinQ-View NK cell subsets (clusters 5 and 6) that resolve modality-exclusive CD8−/CD8+ NK cells should achieve RNA purity scores of 0.9553 and 0.9434 and ADT purity scores of 0.7822 and 0.8279. Write the purity scores and cluster assignments for all methods to a CSV (e.g., figure4_purity_scores.csv) using dplyr and visualize with ggplot2.

## Related tools

- **LinQ-View v0.99** (Provides the joint clustering (L∞ norm fusion of per-modality pairwise distances) and the quantitative purity-score metric applied per cluster per modality) — https://github.com/WilsonImmunologyLab/LinQView
- **ROGUE** (Implements the cluster-purity statistic (Ratio of Global Unshifted Entropy) underlying the purity score; installed as an R dependency from GitHub) — https://github.com/PaulingLiu/ROGUE
- **Seurat** (Runs the default Louvain community-detection clustering on RNA-only and ADT-only modalities that the purity scores are computed over) — https://satijalab.org/seurat/
- **dplyr** (Data wrangling when writing purity scores and cluster assignments to the output CSV) — https://dplyr.tidyverse.org/
- **ggplot2** (Visualization of purity scores per cluster and per modality) — https://ggplot2.tidyverse.org/

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Purity scores are reported for every cluster in BOTH the transcriptome and surface-protein modalities for each of the three clusterings (RNA-only, ADT-only, joint)
- On Dataset 1 (GSE100866), LinQ-View NK clusters 5 and 6 must reproduce RNA purity scores of 0.9553 and 0.9434 and ADT purity scores of 0.7822 and 0.8279
- Only the LinQ-View joint clustering resolves all four modality-exclusive subsets (CD8− and CD8+ NK cells; naive and memory CD4+ T cells); purity scores for unimodal clusterings should reflect their failure to split these
- Output CSV (figure4_purity_scores.csv) contains purity scores and cluster assignments for all three methods and matches Figure 4
- ROGUE dependency installs successfully after the tidyverse package (install.packages('tidyverse')) is present

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency and memory for large datasets
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded; LinQ-View is best suited for large datasets with a limited number of ADT features (<50)
- The purity score addresses the lack of a quantitative cluster-purity metric for CITE-seq data but depends on the upstream clustering; ADT purity scores are lower than RNA purity scores even for high-quality NK clusters (0.78–0.83 vs 0.94–0.96)

## Evidence

- [intro] Purity metric definition: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data"
- [intro] L∞ norm joint distance: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] Default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] Sigmoid filter on ADT distances: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [readme] ROGUE purity concept (README): [evidence span withheld — anchor status: not_found]
- [readme] LinQ-View purpose (README): "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine"
