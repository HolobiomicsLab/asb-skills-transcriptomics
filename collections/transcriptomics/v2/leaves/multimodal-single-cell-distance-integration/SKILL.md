---
name: multimodal-single-cell-distance-integration
description: Use when analyzing CITE-seq data where both RNA and surface protein modalities
  must be integrated — e.g., fusing pairwise distance matrices, clustering mixed modalities,
  or computing a quantitative cluster purity score.
when_to_use_negative:
- Massive datasets such as 170K cells with 228 ADT features — LinQ-View failed on
  this scale whereas Seurat 4 succeeded
- Datasets with a high number of ADT features (≥50) — LinQ-View is best suited for
  large datasets with a limited number of ADT features (e.g., <50)
- Unimodal data with no surface protein modality — the L∞ norm fusion and ADT sigmoid
  filter require both mRNA and protein distances
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: LinQ-View
  role: 'Core toolkit: computes per-modality dissimilarity representations, applies
    the sigmoid ADT filter, fuses distances with the L∞ norm, and computes the purity
    score'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Provides the default Louvain community detection clustering of the joint-distance-defined
    cell populations
  repo: https://satijalab.org/seurat/
- name: dplyr
  role: Tabulates per-cluster purity scores for every dataset and rank value
  repo: https://dplyr.tidyverse.org/
- name: ggplot2
  role: Plots purity-score comparisons across ranks and datasets (Figure 3-style layout)
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:135a9fbb88a444f6447af4bae30696e1359ee2d46cbbf29551f175443340a1d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R v3.6.3
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# multimodal-single-cell-distance-integration

## Summary

Integrates single-cell transcriptome and surface protein (ADT) expression by computing per-modality cell-cell pairwise distances, applying a sigmoid filter to ADT distances, and fusing them with an L∞ norm, then deriving a quantitative purity score for cluster purity assessment in CITE-seq data.

## When to use

Use when analyzing CITE-seq data where both RNA and surface protein modalities must be integrated — e.g., fusing pairwise distance matrices, clustering mixed modalities, or computing a quantitative cluster purity score.

## When NOT to use

- Massive datasets such as 170K cells with 228 ADT features — LinQ-View failed on this scale whereas Seurat 4 succeeded
- Datasets with a high number of ADT features (≥50) — LinQ-View is best suited for large datasets with a limited number of ADT features (e.g., <50)
- Unimodal data with no surface protein modality — the L∞ norm fusion and ADT sigmoid filter require both mRNA and protein distances

## Inputs

- CITE-seq dataset with paired scRNA-seq and ADT (surface protein) counts, e.g. GSE100866 (8,617 cord blood mononuclear cells) or 10x Genomics pbmc_1k_protein_v3
- Per-modality cell-by-feature count matrices (mRNA and ADT)
- Optional: simulated CITE-seq cluster definitions with per-cluster per-feature means and SDs (Table S7)

## Outputs

- Joint cell-cell distance matrix fused with an L∞ norm
- Cell clusters identified via Louvain community detection
- Per-cluster purity scores for transcriptome and ADT modalities (e.g., tabulated via dplyr)
- Comparison plots of purity scores across rank values (e.g., ggplot2, Figure 3-style layout)

## How to apply

Pre-process each modality (cell QC excluding cells with <200 or >2500 detected genes, normalization for both transcriptome and surface protein data, scaling, HVG identification, PCA), then calculate cell-cell pairwise distances for each modality separately. Apply a sigmoid filter to the ADT-modality distance calculation to prevent variations in a single surface protein from dominating the integration, then fuse the modality-specific distance matrices using an L∞ norm model to obtain joint distances. Cluster cells using the default Louvain community detection algorithm implemented in Seurat, and compute LinQ-View's quantitative purity score on both the transcriptome and surface protein modalities. If sweeping the purity-score rank parameter, match rank to ADT feature count: rank 2-3 performs best on a 9-feature dataset, 3-4 on 18 features, and 4-5 on 27 features.

## Related tools

- **LinQ-View** (Core toolkit: computes per-modality dissimilarity representations, applies the sigmoid ADT filter, fuses distances with the L∞ norm, and computes the purity score) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Provides the default Louvain community detection clustering of the joint-distance-defined cell populations) — https://satijalab.org/seurat/
- **dplyr** (Tabulates per-cluster purity scores for every dataset and rank value) — https://dplyr.tidyverse.org/
- **ggplot2** (Plots purity-score comparisons across ranks and datasets (Figure 3-style layout)) — https://ggplot2.tidyverse.org/

## Evaluation signals

- Purity-score rank sweep reproduces the Figure 3 pattern: best-performing rank range is 2-3 on a 9-feature simulation, 3-4 on 18 features, and 4-5 on 27 features
- Simulated cluster ordering matches expectation: clusters 1,2,4,5,6 decrease with variation-feature count, and the single-feature high-variance cluster (cluster 3) scores below cluster 2
- Cell QC invariants hold: all cells with fewer than 200 or more than 2500 detected genes are excluded before distance calculation
- The ADT distance matrix shows attenuation of single-protein outlier influence after the sigmoid filter, so no one surface protein dominates the joint distance
- Joint distances are computed as an L∞ (max-norm) fusion of the mRNA and sigmoid-filtered ADT pairwise distance matrices

## Limitations

- Memory usage and complexity of pairwise distance calculation grow quadratically as dataset size increases, limiting time efficiency
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features, where Seurat 4 succeeded
- Simulated dataset per-cluster per-feature means and SDs are only available in Table S7, not in the main text

## Evidence

- [intro] LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] we introduced a sigmoid filter into the distance calculation of the ADT modality: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [readme] The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression): "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression)"
- [discussion] all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [discussion] LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50): "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
