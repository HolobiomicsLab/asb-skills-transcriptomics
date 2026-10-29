---
name: wilcoxon-rank-sum-deg-testing
description: Use when you need cluster-level DEGs from a single-dataset scRNA-seq
  Seurat object and want per-gene discrimination metrics (AUC, sensitivity, specificity),
  e.g. to benchmark a binary marker method like CDI against a standard Wilcoxon baseline.
when_to_use_negative:
- When the goal is to identify binary (on/off) cluster-discriminating markers with
  maximum cluster specificity — the article's CDI-based findCDIMarkers approach outperformed
  Wilcoxon on specificity (0.95 vs 0.75)
- When the analysis requires modeling variation across biological replicates — the
  article notes existing DE methods for scRNA-seq fail to account for this
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package providing getDEG() for Wilcoxon-based DE testing with sensitivity/specificity
    indices and getExpressedGenes() for feature-space restriction
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Underlying framework; provides the Seurat object, DotPlot rendering, and clustering
    metadata consumed by getDEG()
- name: ggbeeswarm
  role: Bee swarm overlays on sensitivity/specificity boxplots when comparing Wilcoxon
    vs CDI marker metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:5573dcf0d030c8c05e46ac9e52de1500008d8834b5a158184eaaf6e9004d1a8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - getDEG
  - getExpressedGenes
  - ggbeeswarm
  - scMiko
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# wilcoxon-rank-sum-deg-testing

## Summary

Rank-sum based differential expression testing per cell cluster in a Seurat object via scMiko's getDEG(), which returns per-marker AUC, sensitivity, and specificity indices alongside DEG calls, enabling benchmarking of DE methods such as CDI.

## When to use

Use when you need cluster-level DEGs from a single-dataset scRNA-seq Seurat object and want per-gene discrimination metrics (AUC, sensitivity, specificity), e.g. to benchmark a binary marker method like CDI against a standard Wilcoxon baseline.

## When NOT to use

- When the goal is to identify binary (on/off) cluster-discriminating markers with maximum cluster specificity — the article's CDI-based findCDIMarkers approach outperformed Wilcoxon on specificity (0.95 vs 0.75)
- When the analysis requires modeling variation across biological replicates — the article notes existing DE methods for scRNA-seq fail to account for this

## Inputs

- Seurat object with cell-level cluster annotations in metadata (e.g. 'sub_cluster' column, Tyser 2021 so_tyser2021_220621.rds with 1195 cells)
- Vector of expressed genes in the dataset (from getExpressedGenes()) to restrict the testable feature space
- Cluster grouping variable name (e.g. 'sub_cluster')

## Outputs

- Per-cluster DEG table with Wilcoxon significance at FDR < 0.05, plus per-marker AUC, sensitivity, and specificity indices
- Top-50 DEG lists per cluster ranked by AUC
- Seurat DotPlots of top markers per method
- Sensitivity and specificity boxplots with ggbeeswarm overlays, annotated by DE method

## How to apply

Load the Seurat object (e.g. the Tyser 2021 human gastrulation object, 1195 cells, read via scMiko) and visualize the author-annotated 'sub_cluster' populations on the UMAP. Because CDI is computationally intensive, first restrict the feature space with getExpressedGenes(). Run Wilcoxon-based differential expression with getDEG(group_by = 'sub_cluster', return.all = TRUE), which returns per-marker sensitivity and specificity indices in addition to significance; use return.all=TRUE so the AUC ranking column is available. Select the top-ranked DEGs per cluster (top-50 by AUC for Wilcoxon), render Seurat DotPlots of the top markers, and compute significant DEG counts at FDR < 0.05. When benchmarking, match CDI markers (nCDI scores from findCDIMarkers) to the Wilcoxon sensitivity/specificity table by gene-cluster pairs and render sensitivity/specificity boxplots with ggbeeswarm overlays.

## Related tools

- **scMiko** (R package providing getDEG() for Wilcoxon-based DE testing with sensitivity/specificity indices and getExpressedGenes() for feature-space restriction) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Underlying framework; provides the Seurat object, DotPlot rendering, and clustering metadata consumed by getDEG())
- **ggbeeswarm** (Bee swarm overlays on sensitivity/specificity boxplots when comparing Wilcoxon vs CDI marker metrics)

## Examples

```
getDEG(so.query, group_by = 'sub_cluster', return.all = TRUE)
```

## Evaluation signals

- At 5% FDR, Wilcoxon should identify more DEGs than CDI across the evaluated scRNA-seq datasets
- Top-50 Wilcoxon markers ranked by AUC should show cluster specificity around 0.75, versus ~0.95 for CDI's top-50 by nCDI (reported p = 5.4e-3, Fig. 2g,i)
- Per-marker AUC values returned by getDEG(return.all = TRUE) must fall in [0, 1] for every gene-cluster pair
- DotPlots of top-50 AUC-ranked markers should show enriched expression in the target cluster relative to other 'sub_cluster' populations
- Matched gene-cluster pairs between CDI markers and the Wilcoxon sensitivity/specificity table should join cleanly (no unmatched gene-cluster keys)

## Limitations

- Known limitation of existing DE methods for scRNA-seq, including CDI, is failure to account for variation in biological replicates
- Cluster stability and reproducibility issues, especially in single-replicate datasets, can make cluster-DEG assignments unreliable
- Artifact genes highly expressed in a small subset of cells from a single replicate can produce distinct spurious clusters, distorting marker metrics

## Evidence

- [full_text] Wilcoxon-based DE with per-marker discrimination metrics: [evidence span withheld — anchor status: not_found]
- [intro] Article framing of the DE-method validation: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [full_text] Reported CDI vs Wilcoxon comparison result: [evidence span withheld — anchor status: not_found]
- [discussion] Replicate-awareness limitation of DE methods: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception15."
- [full_text] Feature-space restriction step: [evidence span withheld — anchor status: not_found]
