---
name: silhouette-score-evaluation
description: Use when a Seurat/SCISSORS clustering must be assessed for heterogeneity,
  when comparing clusterings across methods with violin plots of per-cell silhouettes,
  or when deciding which broad clusters to recluster for rare subpopulations.
when_to_use_negative:
- When only an external agreement metric is needed (ARI/NMI against known labels)
  and no assessment of within-cluster heterogeneity or reclustering targets is intended
- When the input is raw counts or a feature table that has not been normalized and
  reduced to PCA space — silhouette scores are only meaningful on the embedding (dims
  from RunPCA)
- When clusters are already known to be homogeneous fine-grained subclusters (e.g.
  a validated 148-cell rare subcluster) — silhouette evaluation targets broad, heterogeneous
  parent clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: SCISSORS
  role: Provides ComputeSilhouetteScores for per-cell silhouette estimation on Seurat
    objects and uses silhouette scores to drive semi-supervised reclustering of heterogeneous
    clusters
  repo: https://github.com/jr-leary7/SCISSORS
- name: cluster
  role: cluster::silhouette computes silhouettes on the CosineDist cosine-distance
    matrix of PCA embeddings for SAFE and GiniClust3 clusterings
- name: ggplot2
  role: Renders violin plots of per-cell silhouette score distributions per method
    (draw_quantiles = 0.5)
- name: ggsignif
  role: Adds paired wilcox.test significance bars between methods on the silhouette
    violin plot
- name: Seurat
  role: Supplies the Seurat object, PCA embeddings (Embeddings(pbmc, "pca")), and
    Louvain clustering resolutions evaluated via silhouette/ARI/NMI
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:93643db8f62821656268313723826931c6e350707d7a2eeee84ad774e20a80c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - CONICSmat
  - R
  - SAFEclustering
  - SCISSORS
  - Seurat
  - SeuratData
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# silhouette-score-evaluation

## Summary

Per-cell silhouette scoring of scRNA-seq clusterings in PCA (or cosine-distance) space to quantify cluster heterogeneity, compare candidate clusterings (SCISSORS vs Seurat vs SAFE vs GiniClust3), and flag broad clusters that warrant semi-supervised reclustering.

## When to use

Use when a Seurat/SCISSORS clustering must be assessed for heterogeneity, when comparing clusterings across methods with violin plots of per-cell silhouettes, or when deciding which broad clusters to recluster for rare subpopulations.

## When NOT to use

- When only an external agreement metric is needed (ARI/NMI against known labels) and no assessment of within-cluster heterogeneity or reclustering targets is intended
- When the input is raw counts or a feature table that has not been normalized and reduced to PCA space — silhouette scores are only meaningful on the embedding (dims from RunPCA)
- When clusters are already known to be homogeneous fine-grained subclusters (e.g. a validated 148-cell rare subcluster) — silhouette evaluation targets broad, heterogeneous parent clusters

## Inputs

- Preprocessed Seurat object with PCA embeddings (e.g. PBMC3k prepared with PrepareData(n.HVG = 4000, n.PC = 15))
- Cluster label vectors from one or more clustering methods (SCISSORS, Seurat Louvain, SAFE, GiniClust3)
- PCA embedding matrix for computing CosineDist cosine distances when using cluster::silhouette directly
- Simulated pancreas dataset (996 cells, 15,270 genes) with 7 ground-truth cluster labels for benchmarking against ARI/NMI

## Outputs

- Per-cell silhouette score data frame (cluster assignment + silhouette value per cell)
- Per-cluster median silhouette summary used to select heterogeneous clusters for reclustering
- ggplot2 violin plot of per-method silhouette distributions with ggsignif wilcox.test significance bars
- Metrics table combining silhouette, ARI, and NMI across methods / Louvain resolutions (saved as PDF figures)

## How to apply

Run ComputeSilhouetteScores on the Seurat object with avg = FALSE to obtain per-cell silhouette scores, then summarize them per broad cluster and inspect median values: clusters with low median silhouette (e.g. PBMC3k clusters 0, 1, & 3) are targets for SCISSORS reclustering. For clusterings produced outside Seurat (SAFE, GiniClust3) or for cosine-based comparisons, compute cluster::silhouette on a CosineDist cosine-distance matrix built from the PCA embeddings. Compare methods by plotting per-cell silhouette distributions as ggplot2 violin plots (draw_quantiles = 0.5), adding ggsignif significance bars from paired wilcox.test comparisons, and require the method of interest (e.g. SCISSORS) to show the highest median silhouette with statistically significant differences versus alternatives. For resolution sweeps (Louvain resolution 0.5–2.0), report silhouette alongside ARI and NMI so that the optimal (lowest) resolution is confirmed by all three metrics rather than silhouette alone.

## Related tools

- **SCISSORS** (Provides ComputeSilhouetteScores for per-cell silhouette estimation on Seurat objects and uses silhouette scores to drive semi-supervised reclustering of heterogeneous clusters) — https://github.com/jr-leary7/SCISSORS
- **cluster** (cluster::silhouette computes silhouettes on the CosineDist cosine-distance matrix of PCA embeddings for SAFE and GiniClust3 clusterings)
- **ggplot2** (Renders violin plots of per-cell silhouette score distributions per method (draw_quantiles = 0.5))
- **ggsignif** (Adds paired wilcox.test significance bars between methods on the silhouette violin plot)
- **Seurat** (Supplies the Seurat object, PCA embeddings (Embeddings(pbmc, "pca")), and Louvain clustering resolutions evaluated via silhouette/ARI/NMI)

## Examples

```
sil_score_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE)
```

## Evaluation signals

- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE) returns a data frame with one silhouette value per cell that can be summarized per cluster
- On the simulated 996-cell pancreas dataset, SCISSORS must attain silhouette = 0.693 (second highest) alongside ARI = 0.987 and NMI = 0.985, and be the only method separating the two rare clusters
- On PBMC3k, the four-method violin plot must show SCISSORS with the highest median silhouette and statistically significant paired wilcox.test differences versus Seurat, SAFE, and GiniClust3 (ggsignif bars)
- In the Louvain resolution sweep (0.5–2.0), silhouette must peak at the lowest resolution (0.5), consistent with ARI and NMI, confirming that increasing resolution does not improve clustering
- Heterogeneous parent clusters (e.g. merged PDAC epithelial Clusters 5 and 9) must show low per-cell silhouette scores on cosine distance of PCA embeddings before reclustering, validating the choice to recluster

## Limitations

- Silhouette scores depend on the embedding and distance metric choice — cosine distance on PCA embeddings is used for external methods, PCA-space distances via cluster::silhouette for benchmarks, so scores are not directly comparable across setups without care
- Even small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, so silhouette-based conclusions (e.g. resolution sweep rankings) are sensitive to preprocessing parameters such as n.HVG, n.PC, and random.seed
- No discussion-section text was available in the provided document, so additional caveats or reproducibility gaps could not be extracted from the discussion

## Evidence

- [abstract] silhouette scoring for heterogeneity estimation: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [methods] per-cell silhouette computation: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [methods] cosine distance on PCA embeddings: "cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))"
- [readme] README low-median clusters as reclustering targets: [evidence span withheld — anchor status: external_doc]
- [results] task_002 silhouette computation on PCA-space distances: [evidence span withheld — anchor status: not_found]
