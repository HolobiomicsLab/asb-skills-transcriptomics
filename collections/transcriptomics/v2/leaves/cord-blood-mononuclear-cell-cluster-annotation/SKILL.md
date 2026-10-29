---
name: cord-blood-mononuclear-cell-cluster-annotation
description: Use when a CITE-seq dataset of cord blood mononuclear cells needs cluster-level
  annotation of immune subsets (e.g., NK and CD4+ T cells) and multimodal integration
  quality must be benchmarked against methods like MOFA2, Seurat 4, and SNF.
when_to_use_negative:
- Datasets with a high number of ADT features or massive cell counts (e.g., 170K cells
  with 228 ADT features) — LinQ-View's pairwise distance computation scales quadratically
  and failed there, whereas Seurat 4 succeeded
- Large datasets with many ADT features where Seurat 4 is the recommended option for
  both performance and time efficiency
- Data lacking a surface-protein (ADT) modality, since the L-infinity-norm fusion
  depends on per-modality distance matrices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: LinQ-View
  role: 'Primary multimodal integration toolkit: fuses transcriptome and ADT pairwise
    distance matrices with an L-infinity norm and clusters via Louvain community detection'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat 4
  role: Comparator multimodal integration method (WNN) used at Louvain resolution
    0.9; best option for massive CITE-seq datasets with many ADT features
  repo: https://satijalab.org/seurat/
- name: MOFA2
  role: Comparator factor-analysis method run at resolutions 0.9 and 1.5 to test subset
    recovery
- name: SNF
  role: Comparator similarity network fusion method run at resolution 0.9; slowest
    method (3-4 h)
- name: ROGUE
  role: Quantitative cluster purity assessment (Ratio of Global Unshifted Entropy)
    to check whether recovered clusters are pure cell populations
  repo: https://github.com/PaulingLiu/ROGUE
- name: R v3.6.3
  role: Runtime environment for LinQ-View, Seurat, and ROGUE analyses
  repo: https://www.r-project.org/
- name: ggplot2
  role: Visualization of the side-by-side subset-recovery comparison figure
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:b1eb465c10804a2c7e5e280cfada705bacd5fd10fe83237b7b18b71fd3bd3546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - MOFA2
  - R v3.6.3
  - ROGUE
  - SNF
  - Seurat 4
  - ggplot2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cord-blood-mononuclear-cell-cluster-annotation

## Summary

Annotates cell heterogeneity (NK and CD4+ T cell subsets) in the 8,617-cell cord blood mononuclear cell (CBMC) CITE-seq benchmark (GSE100866) by clustering integrated transcriptome and surface-protein modalities at Louvain resolution 0.9 with LinQ-View and comparator multimodal methods.

## When to use

Use when a CITE-seq dataset of cord blood mononuclear cells needs cluster-level annotation of immune subsets (e.g., NK and CD4+ T cells) and multimodal integration quality must be benchmarked against methods like MOFA2, Seurat 4, and SNF.

## When NOT to use

- Datasets with a high number of ADT features or massive cell counts (e.g., 170K cells with 228 ADT features) — LinQ-View's pairwise distance computation scales quadratically and failed there, whereas Seurat 4 succeeded
- Large datasets with many ADT features where Seurat 4 is the recommended option for both performance and time efficiency
- Data lacking a surface-protein (ADT) modality, since the L-infinity-norm fusion depends on per-modality distance matrices

## Inputs

- CITE-seq count matrices for transcriptome and surface-protein (ADT) modalities from GSE100866 (8,617 cord blood mononuclear cells, Stoeckius et al. 2017)
- Louvain clustering resolution parameter (0.9 primary; 1.5 for MOFA2 sensitivity run)

## Outputs

- Cell cluster assignments with NK and CD4+ T cell subset annotations at resolution 0.9
- Side-by-side comparison figure (Figure 6-style) of LinQ-View vs SNF, MOFA2, Seurat 4 subset recovery
- runtime_comparison.csv recording elapsed runtimes per method (LinQ-View ~30 s, Seurat 4 ~20 s, MOFA2 ~6 min, SNF 3-4 h)

## How to apply

Load Dataset 1 (8,617 CBMCs, GSE100866) and run the standard pre-processing chain: cell QC (exclude cells with <200 or >2500 detected genes), normalization and scaling of both transcriptome and surface-protein modalities, HVG identification, and PCA. Run LinQ-View, which fuses per-modality cell-cell pairwise distance matrices with an L-infinity norm (a sigmoid filter is applied to the ADT distance calculation) and clusters via Seurat-implemented Louvain community detection at resolution 0.9, then annotate recovered NK and CD4+ T cell subsets. In parallel, run Seurat 4 multimodal integration, SNF, and MOFA2 at the same resolution 0.9 (and MOFA2 at 1.5 as a sensitivity check), recording which subsets each recovers. Expected result: LinQ-View and Seurat 4 correctly separate two NK cell subsets at resolution 0.9 while MOFA2 and SNF fail; MOFA2 resolves them only at resolution 1.5; CD4+ T cell subsets are recovered as 2 (LinQ-View), 4 (MOFA2), and 5 (Seurat 4) clusters. Optionally score cluster purity with ROGUE, using a technology-dependent pre-defined K parameter.

## Related tools

- **LinQ-View** (Primary multimodal integration toolkit: fuses transcriptome and ADT pairwise distance matrices with an L-infinity norm and clusters via Louvain community detection) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat 4** (Comparator multimodal integration method (WNN) used at Louvain resolution 0.9; best option for massive CITE-seq datasets with many ADT features) — https://satijalab.org/seurat/
- **MOFA2** (Comparator factor-analysis method run at resolutions 0.9 and 1.5 to test subset recovery)
- **SNF** (Comparator similarity network fusion method run at resolution 0.9; slowest method (3-4 h))
- **ROGUE** (Quantitative cluster purity assessment (Ratio of Global Unshifted Entropy) to check whether recovered clusters are pure cell populations) — https://github.com/PaulingLiu/ROGUE
- **R v3.6.3** (Runtime environment for LinQ-View, Seurat, and ROGUE analyses) — https://www.r-project.org/
- **ggplot2** (Visualization of the side-by-side subset-recovery comparison figure) — https://ggplot2.tidyverse.org/

## Examples

```
devtools::install_github("WilsonImmunologyLab/LinQView"); LinQView::RunLinQView(object, resolution = 0.9, reduction = "umap")
```

## Evaluation signals

- LinQ-View and Seurat 4 each recover two NK cell subsets at Louvain resolution 0.9
- SNF and MOFA2 (at 0.9) fail to separate any NK cell subsets; MOFA2 succeeds only when re-run at resolution 1.5
- CD4+ T cell subsets are recovered as 2, 4, and 5 clusters by LinQ-View, MOFA2, and Seurat 4, respectively
- Measured runtimes reproduce the ordering LinQ-View/Seurat 4 (tens of seconds) << MOFA2 (~6 min) << SNF (3-4 h)
- Cell QC removes all cells with fewer than 200 or more than 2500 detected genes before clustering

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency on large datasets
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded; LinQ-View is best suited for datasets with a limited number of ADT features (e.g., <50)
- The ROGUE purity score requires a pre-defined parameter K that is technology-dependent but not specified in the text
- Subset-recovery outcomes are sensitive to Louvain resolution (MOFA2 fails at 0.9 but succeeds at 1.5)

## Evidence

- [intro] LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [discussion] a dataset of 8,617 cord blood mononuclear cells (CBMCs), produced with CITE-seq (Stoeckius et al., 2017). NCBI GEO accession number is GSE100866: "a dataset of 8,617 cord blood mononuclear cells (CBMCs), produced with CITE-seq (Stoeckius et al., 2017). NCBI GEO accession number is GSE100866"
- [discussion] Considering both performance and time efficiency, Seurat 4 is the best option for massive CITE-seq datasets with a high number of ADT features: "Considering both performance and time efficiency, Seurat 4 is the best option for massive CITE-seq datasets with a high number of ADT features"
- [discussion] The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] we introduced a sigmoid filter into the distance calculation of the ADT modality: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [readme] combining dissimilarity representations may help emphasize different information from each modality to maximally capture and infer cell heterogeneity: "combining dissimilarity representations may help emphasize different information from each modality to maximally capture and infer cell heterogeneity"
