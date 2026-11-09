---
name: single-cell-cluster-marker-identification
description: Use when a Seurat/scMiko scRNA-seq object has author- or algorithm-defined
  cluster labels (e.g. 'sub_cluster') and you need cluster-specific DEGs, or want
  to benchmark CDI (binary DEG) markers against Wilcoxon markers for specificity.
when_to_use_negative:
- The analysis goal requires DE methods that account for variation in biological replicates
  — the CDI approach, like existing scRNA-seq DE methods, fails to account for replicate-level
  variation
- Clusters come from a single-replicate dataset where spurious clusters from artifact
  genes cannot be distinguished from real biology
- Input is raw count data without cluster labels — cell-type annotation (e.g. Miko
  scoring or sc-type) must precede marker identification
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package providing findCDIMarkers(), getDEG(), and getExpressedGenes() for
    CDI and Wilcoxon marker identification
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework scMiko is built on; DotPlot rendering of top markers per method
- name: ggbeeswarm
  role: Bee-swarm overlays on sensitivity and specificity boxplots comparing CDI vs
    Wilcoxon markers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:a246468553ff5240c12ac99859bcbf25234ea519e93a6637beebab16f23a3442
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-cluster-marker-identification

## Summary

Identify cell-type/cluster-specific marker genes in scRNA-seq data by running co-dependency index (CDI) and/or Wilcoxon rank-sum differential expression per cluster, then rank and compare top markers by cluster-discriminating specificity (nCDI vs AUC).

## When to use

Use when a Seurat/scMiko scRNA-seq object has author- or algorithm-defined cluster labels (e.g. 'sub_cluster') and you need cluster-specific DEGs, or want to benchmark CDI (binary DEG) markers against Wilcoxon markers for specificity.

## When NOT to use

- The analysis goal requires DE methods that account for variation in biological replicates — the CDI approach, like existing scRNA-seq DE methods, fails to account for replicate-level variation
- Clusters come from a single-replicate dataset where spurious clusters from artifact genes cannot be distinguished from real biology
- Input is raw count data without cluster labels — cell-type annotation (e.g. Miko scoring or sc-type) must precede marker identification

## Inputs

- Seurat object with cluster annotations in a metadata column (e.g. so_tyser2021_220621.rds, Tyser 2021 human gastrulation, 1195 cells)
- Gene expression feature list restricted to expressed genes from getExpressedGenes()
- Cluster label metadata field (e.g. 'sub_cluster')

## Outputs

- Per gene-cluster pair nCDI scores from findCDIMarkers()
- Wilcoxon DEG table with per-marker sensitivity and specificity indices from getDEG()
- Top-50 marker lists per cluster per method (nCDI-ranked vs AUC-ranked)
- Seurat DotPlots of top markers per method
- Significant DEG counts at FDR < 0.05 and cluster specificity statistics with p-values

## How to apply

First restrict the feature space to genes expressed within the dataset using getExpressedGenes(), since CDI is computationally intensive over the full transcriptome. Run CDI-based DE with findCDIMarkers(features.x = 'sub_cluster', features.y = expressed genes) to obtain normalized CDI (nCDI) scores per gene-cluster pair, and run Wilcoxon-based DE with getDEG(group_by = 'sub_cluster', return.all = TRUE), which also returns per-marker sensitivity and specificity indices. Select top-ranked DEGs per cluster (e.g. top-50 by nCDI for CDI, top-50 by AUC for Wilcoxon), render Seurat DotPlots of the top markers, and compute significant DEG counts at FDR < 0.05. Match CDI markers to the Wilcoxon sensitivity/specificity table by gene-cluster pairs, annotate the method used, and render sensitivity/specificity boxplots with ggbeeswarm overlays. Correct application reproduces the pattern that CDI identifies fewer significant DEGs than Wilcoxon but its top-50 markers show higher cluster specificity.

## Related tools

- **scMiko** (R package providing findCDIMarkers(), getDEG(), and getExpressedGenes() for CDI and Wilcoxon marker identification) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework scMiko is built on; DotPlot rendering of top markers per method)
- **ggbeeswarm** (Bee-swarm overlays on sensitivity and specificity boxplots comparing CDI vs Wilcoxon markers)

## Examples

```
expressed.genes <- getExpressedGenes(so.query); cdi.res <- findCDIMarkers(so.query, features.x = 'sub_cluster', features.y = expressed.genes); deg.res <- getDEG(so.query, group_by = 'sub_cluster', return.all = TRUE)
```

## Evaluation signals

- At 5% FDR, CDI identifies fewer significant DEGs than Wilcoxon across the datasets
- Top-50 CDI markers achieve higher cluster specificity than Wilcoxon top-50 markers (reported: 0.95 vs 0.75, p = 5.4e-3)
- Every selected top marker maps to a valid gene-cluster pair in both the nCDI table and the Wilcoxon sensitivity/specificity table
- DotPlots of top-50 markers per cluster show expected expression restriction to the assigned cluster for each method
- Feature space was restricted to genes expressed within the dataset before running CDI (CDI is computationally intensive otherwise)

## Limitations

- A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception
- CDI is computationally intensive, requiring feature-space restriction to expressed genes
- Marker/cluster quality depends on the clustering configuration; specificity-based resolution selection is limited by stability and reproducibility of clusters, especially in single-replicate datasets

## Evidence

- [results] CDI identifies fewer DEGs with higher specificity than Wilcoxon: [evidence span withheld — anchor status: not_found]
- [intro] CDI validated as binary DEG method: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [discussion] CDI does not account for biological replicates: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception"
- [methods] scMiko installation: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
- [readme] scMiko built on Seurat framework: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework, and offers existing and novel single-cell analytic"
