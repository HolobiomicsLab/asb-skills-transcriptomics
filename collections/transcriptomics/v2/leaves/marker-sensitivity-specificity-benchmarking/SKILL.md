---
name: marker-sensitivity-specificity-benchmarking
description: Use when comparing DE methods (e.g., CDI vs Wilcoxon) for cell-type-specific
  binary DEGs across scRNA-seq datasets, and you need DEG counts plus per-marker cluster-discriminating
  sensitivity/specificity benchmarking.
when_to_use_negative:
- Datasets with multiple biological replicates where DE methods that model replicate
  variation are required — CDI, like other scRNA-seq DE methods, does not account
  for variation in biological replicates.
- Single-replicate datasets where clusters may be unstable or spurious (artifact genes
  in a small subset of cells from one replicate can create distinct clusters), undermining
  specificity estimates as a benchmark signal.
- Analysis goals beyond binary cell-type-specific DEG identification (e.g., graded,
  non-binary differential expression) since CDI is validated specifically for binary
  DEGs (bDEGs).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMiko
  role: Provides findCDIMarkers(), getDEG(), and getExpressedGenes() used to run the
    CDI and Wilcoxon DE comparisons and match markers by gene-cluster pairs
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework underlying the Seurat object and DotPlot rendering of top markers
    per method
  repo: https://satijalab.org/seurat/
- name: ggbeeswarm
  role: Beeswarm overlays on the sensitivity and specificity boxplots comparing CDI
    and Wilcoxon markers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:435b529515261b9e0956703e6bf8debe4f7cfb49a5cd3e53756ab92f997f9232
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# marker-sensitivity-specificity-benchmarking

## Summary

Benchmarks the CDI differential expression method against Wilcoxon for identifying cell-type-specific binary DEGs in scRNA-seq, comparing DEG counts at FDR < 0.05 and top-50 marker cluster specificity via per-marker sensitivity/specificity indices.

## When to use

Use when comparing DE methods (e.g., CDI vs Wilcoxon) for cell-type-specific binary DEGs across scRNA-seq datasets, and you need DEG counts plus per-marker cluster-discriminating sensitivity/specificity benchmarking.

## When NOT to use

- Datasets with multiple biological replicates where DE methods that model replicate variation are required — CDI, like other scRNA-seq DE methods, does not account for variation in biological replicates.
- Single-replicate datasets where clusters may be unstable or spurious (artifact genes in a small subset of cells from one replicate can create distinct clusters), undermining specificity estimates as a benchmark signal.
- Analysis goals beyond binary cell-type-specific DEG identification (e.g., graded, non-binary differential expression) since CDI is validated specifically for binary DEGs (bDEGs).

## Inputs

- Seurat object with author-annotated cell-type clusters (e.g., so_tyser2021_220621.rds, 1195 cells, 'sub_cluster' metadata column)
- Gene expression matrix restricted to genes expressed within the dataset (output of getExpressedGenes())
- Cluster metadata grouping variable (e.g., 'sub_cluster')

## Outputs

- Normalized CDI (nCDI) scores per gene-cluster pair from findCDIMarkers
- Wilcoxon DEG table with per-marker sensitivity and specificity indices from getDEG
- Seurat DotPlots of top-50 markers per method (top-50 by nCDI vs top-50 by AUC)
- Significant DEG counts per cluster at FDR < 0.05 per method
- Sensitivity and specificity boxplots with ggbeeswarm overlays comparing CDI vs Wilcoxon markers

## How to apply

Load the author-annotated Seurat object (e.g., the Tyser 2021 human gastrulation dataset with 1195 cells and 'sub_cluster' annotations) via scMiko and restrict the feature space to genes expressed in the dataset with getExpressedGenes(), since CDI is computationally intensive. Run CDI-based DE with findCDIMarkers(features.x = 'sub_cluster', features.y = expressed genes) to obtain normalized CDI (nCDI) scores per gene-cluster pair, and Wilcoxon-based DE with getDEG(group_by = 'sub_cluster', return.all = TRUE), which also returns per-marker sensitivity and specificity indices. Select top-ranked DEGs per cluster (top-50 by nCDI for CDI, top-50 by AUC for Wilcoxon), render Seurat DotPlots of the top markers, and compute significant DEG counts at FDR < 0.05. Match CDI markers to the Wilcoxon sensitivity/specificity table by gene-cluster pairs, annotate the method, and render sensitivity and specificity boxplots with ggbeeswarm overlays. The validation pattern is that CDI identifies fewer DEGs at 5% FDR but higher cluster-discriminating specificity (0.95 vs. 0.75, p = 5.4e-3).

## Related tools

- **scMiko** (Provides findCDIMarkers(), getDEG(), and getExpressedGenes() used to run the CDI and Wilcoxon DE comparisons and match markers by gene-cluster pairs) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework underlying the Seurat object and DotPlot rendering of top markers per method) — https://satijalab.org/seurat/
- **ggbeeswarm** (Beeswarm overlays on the sensitivity and specificity boxplots comparing CDI and Wilcoxon markers)

## Examples

```
findCDIMarkers(object = so.query, features.x = 'sub_cluster', features.y = getExpressedGenes(so.query)); getDEG(object = so.query, group_by = 'sub_cluster', return.all = TRUE)
```

## Evaluation signals

- At 5% FDR, CDI identifies fewer significant DEGs than Wilcoxon across the dataset(s).
- Top-50 CDI markers achieve higher cluster specificity than top-50 Wilcoxon (AUC-ranked) markers, reproducing the reported 0.95 vs. 0.75 difference (p = 5.4e-3).
- Every CDI marker gene-cluster pair is matched to a row in the Wilcoxon sensitivity/specificity table with a method annotation column.
- Each cluster has a rendered DotPlot of its top-50 markers for both methods (nCDI-ranked and AUC-ranked).
- findCDIMarkers returns nCDI values for every gene-cluster pair over the expressed-gene feature space only.

## Limitations

- The CDI approach, like existing DE methods for scRNA-seq, does not account for variation in biological replicates.
- Specificity-based benchmarking is limited by the stability and reproducibility of clusters, especially in single-replicate datasets, where technical artifacts and real biology are hard to distinguish.
- CDI is computationally intensive, so the feature space must be pre-restricted to genes expressed within the dataset before running findCDIMarkers.

## Evidence

- [intro] we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs): "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [discussion] CDI limitation discussion: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception"
- [readme] scMiko installation from README: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
- [methods] scMiko install call from methods: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
