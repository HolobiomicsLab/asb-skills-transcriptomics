---
name: wilcoxon-ranked-significance-testing
description: Use when comparing clustering quality between two or more single-cell
  clustering solutions — e.g., per-cell silhouette scores from SCISSORS, Seurat, SAFE,
  or GiniClust3 — and you need paired significance testing on the score distributions.
when_to_use_negative:
- Cluster-level averaged silhouette values are used instead of per-cell scores (avg
  = TRUE or cluster::silhouette with average=TRUE), since pairing is lost and the
  test loses power for rare subclusters
- Clusterings partition different sets of cells, so no valid cell-level pairing exists
  between the score distributions
- The goal is differential expression testing between cell clusters rather than comparison
  of clustering-quality metrics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: cluster
  role: Provides silhouette() to compute per-cell silhouette scores on cosine-distance
    matrices for SAFE and GiniClust3 clusterings
- name: SCISSORS
  role: Provides ComputeSilhouetteScores(avg = FALSE) to compute per-cell silhouette
    scores for SCISSORS- and Seurat-clustered Seurat objects
  repo: https://github.com/jr-leary7/SCISSORS
- name: ggplot2
  role: Renders violin plots of silhouette score distributions with draw_quantiles
    = 0.5 for the method comparison
- name: ggsignif
  role: Annotates paired wilcox.test comparisons as significance bars between the
    violin plots
- name: SAFEclustering
  role: One of the competing clusterings (SC3 + Seurat + tSNE ensemble) whose silhouette
    scores are tested against SCISSORS via wilcox.test
- name: giniclust3
  role: Rare-cell clustering method (GiniClust3) whose silhouette scores are tested
    against SCISSORS via wilcox.test
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:dd8154b98ffbd146a78b64fd78731d35cf1b09b8371abcceac2b91bf8f26c7da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SAFEclustering
  - SCISSORS
  - SeuratData
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# wilcoxon-ranked-significance-testing

## Summary

Applies paired Wilcoxon rank-sum tests (wilcox.test) to per-cell silhouette score distributions to statistically compare the clustering quality of competing scRNA-seq clusterings (e.g., SCISSORS vs. Seurat, SAFE, GiniClust3), with results annotated as significance levels on violin plots via ggsignif.

## When to use

Use when comparing clustering quality between two or more single-cell clustering solutions — e.g., per-cell silhouette scores from SCISSORS, Seurat, SAFE, or GiniClust3 — and you need paired significance testing on the score distributions.

## When NOT to use

- Cluster-level averaged silhouette values are used instead of per-cell scores (avg = TRUE or cluster::silhouette with average=TRUE), since pairing is lost and the test loses power for rare subclusters
- Clusterings partition different sets of cells, so no valid cell-level pairing exists between the score distributions
- The goal is differential expression testing between cell clusters rather than comparison of clustering-quality metrics

## Inputs

- Per-cell silhouette score dataframes (e.g., sils_SCISSORS, sils_Seurat from ComputeSilhouetteScores with avg = FALSE)
- Silhouette scores from cluster::silhouette computed on CosineDist cosine-distance matrices of PCA embeddings (for SAFE and GiniClust3 clusterings)
- Paired cell-level dataframe with columns: method/factor label, per-cell silhouette score

## Outputs

- Set of paired wilcox.test results (test statistics and p-values) for each method-vs-reference comparison
- A ggplot2 violin plot (draw_quantiles = 0.5) with ggsignif significance bars mapping mapped significance levels between methods
- A four-method comparison dataframe of silhouette scores keyed by clustering method

## How to apply

First compute per-cell (not per-cluster-averaged) silhouette scores for each clustering: for SCISSORS and Seurat objects use ComputeSilhouetteScores(obj, avg = FALSE); for SAFE and GiniClust3 clusterings use cluster::silhouette on a cosine-distance matrix built with CosineDist(input = Embeddings(pbmc, "pca")). Assemble a single dataframe of (method, cell, silhouette score) so comparisons are paired at the cell level. Run paired two-sided wilcox.test comparing the reference method (e.g., SCISSORS) against each competitor (GiniClust3, Seurat, SAFE). Render the score distributions as ggplot2 violin plots with draw_quantiles = 0.5 (median line) and map the wilcox.test p-values to significance levels (e.g., * < 0.05, ** < 0.01) on ggsignif bars. Validation criterion from the PBMC3k comparison: the reference clustering must show the highest median silhouette score AND statistically significant wilcox.test differences versus all competitors shown by the ggsignif bars.

## Related tools

- **cluster** (Provides silhouette() to compute per-cell silhouette scores on cosine-distance matrices for SAFE and GiniClust3 clusterings)
- **SCISSORS** (Provides ComputeSilhouetteScores(avg = FALSE) to compute per-cell silhouette scores for SCISSORS- and Seurat-clustered Seurat objects) — https://github.com/jr-leary7/SCISSORS
- **ggplot2** (Renders violin plots of silhouette score distributions with draw_quantiles = 0.5 for the method comparison)
- **ggsignif** (Annotates paired wilcox.test comparisons as significance bars between the violin plots)
- **SAFEclustering** (One of the competing clusterings (SC3 + Seurat + tSNE ensemble) whose silhouette scores are tested against SCISSORS via wilcox.test)
- **giniclust3** (Rare-cell clustering method (GiniClust3) whose silhouette scores are tested against SCISSORS via wilcox.test)

## Examples

```
wilcox.test(sils_SCISSORS$score, sils_Seurat$score, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- Per-cell silhouette scores were used (ComputeSilhouetteScores called with avg = FALSE), so the paired test is performed at the cell level
- The comparison is paired (paired = TRUE wilcox.test) across the same cells under each clustering
- The violin plot shows draw_quantiles = 0.5 median lines, and the reference method (SCISSORS) has the highest median silhouette score
- ggsignif bars display mapped significance levels for all paired wilcox.test comparisons (SCISSORS vs. GiniClust3, Seurat, and SAFE), and SCISSORS differences are statistically significant
- The cosine-distance matrix for SAFE/GiniClust3 scores was built from PCA embeddings via CosineDist(input = Embeddings(pbmc, "pca")) using cluster::silhouette

## Limitations

- Silhouette scores depend on the distance metric and embedding used (cosine distance on PCA embeddings for SAFE/GiniClust3 vs. SCISSORS' internal scoring), so cross-method comparisons mix scoring schemes unless carefully aligned
- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, making significance claims sensitive to preprocessing choices (e.g., n.HVG, n.PC, resolution, random.seed = 629)
- The comparison as documented was performed on a single dataset (PBMC3k, ~2,700 cells), so significance results may not generalize to larger or rarer-cell populations

## Evidence

- [abstract] SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells"
- [methods] ComputeSilhouetteScores per-cell: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [methods] CosineDist on PCA embeddings: "cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))"
- [readme] README silhouette distribution visualization: [evidence span withheld — anchor status: external_doc]
- [abstract] small differences in parameter choice alter clustering results: "even small differences in analysis and parameter choosing can greatly alter clustering results"
