---
name: preranked-gene-enrichment-analysis
description: Use when annotated cell type identities must be assigned to clusters
  of an integrated scRNA-seq dataset (e.g., human HSPCs) using curated signature gene
  sets rather than manual marker inspection.
when_to_use_negative:
- The system of interest lacks relevant curated signature gene sets — the article
  notes the user would have to compile and curate customized gene sets, since only
  75 hematopoiesis gene sets are provided
- The input is raw count matrices or unclustered cells — clusters must already be
  identified before pre-ranking marker genes
- Manual inspection of a handful of known marker genes on a single small dataset is
  sufficient and no gene-set enrichment framework is desired
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R package providing identifyGSEAPrerankedGenes_for_all_clusters, get_gmtGeneSets,
    listAvailableGeneSets, Run_fGSEA_for_multiple_comparisons, and plot_heatmap_for_fGSEA_all_clusters
    to run the preranked GSEA annotation workflow
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Execution environment for the SingCellaR package and the fGSEA annotation
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:c855c6e760a70ac3c030d4b7f6ba3aba1a302f1eb16e5df6dc825f597457f2e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# preranked-gene-enrichment-analysis

## Summary

Objective cell type annotation for scRNA-seq clusters by pre-ranking cluster marker genes by log2 fold-change and running fast GSEA (fGSEA) against curated GMT gene sets, then visualizing normalized enrichment scores (NES) as a cluster-by-gene-set heatmap to assign identities and cell states.

## When to use

Use when annotated cell type identities must be assigned to clusters of an integrated scRNA-seq dataset (e.g., human HSPCs) using curated signature gene sets rather than manual marker inspection.

## When NOT to use

- The system of interest lacks relevant curated signature gene sets — the article notes the user would have to compile and curate customized gene sets, since only 75 hematopoiesis gene sets are provided
- The input is raw count matrices or unclustered cells — clusters must already be identified before pre-ranking marker genes
- Manual inspection of a handful of known marker genes on a single small dataset is sufficient and no gene-set enrichment framework is desired

## Inputs

- Clustered SingCellaR object with cluster.type defined (integrated scRNA-seq dataset, e.g., human HSPC clusters)
- Curated GMT file of signature gene sets (75 hematopoiesis gene sets curated from previous studies)
- Pre-ranked gene lists per cluster (log2FC-ordered differential expression tables from identifyGSEAPrerankedGenes_for_all_clusters)

## Outputs

- fGSEA result table of NES values and enrichment p-values per cluster per gene set
- Cluster-by-gene-set NES heatmap figure (fgsea_nes_heatmap.png)
- Cell type identity and cell state annotation per cluster

## How to apply

On a clustered SingCellaR object, call identifyGSEAPrerankedGenes_for_all_clusters(object, cluster.type, fishers_exact_test, min.expFraction, min.log2FC) to pre-rank differentially expressed genes for every cluster by log2 fold-change. Verify the gene-set universe against the curated 75 hematopoiesis GMT file with get_gmtGeneSets(gmt.file) / listAvailableGeneSets(gmt.file). Pass the list of pre-ranked gene tables to Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed) to compute NES and enrichment p-values for each cluster against each gene set. Finally plot the NES matrix with plot_heatmap_for_fGSEA_all_clusters and save as fgsea_nes_heatmap.png; annotate each cluster's identity and cell state from the degree and direction of enrichment. The rationale is that enrichment directions consistent with the log2FC pre-ranking provide a systematic, objective basis for annotation instead of subjective marker-by-marker inspection.

## Related tools

- **SingCellaR** (R package providing identifyGSEAPrerankedGenes_for_all_clusters, get_gmtGeneSets, listAvailableGeneSets, Run_fGSEA_for_multiple_comparisons, and plot_heatmap_for_fGSEA_all_clusters to run the preranked GSEA annotation workflow) — https://github.com/supatt-lab/SingCellaR
- **R** (Execution environment for the SingCellaR package and the fGSEA annotation workflow)

## Examples

```
Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed); plot_heatmap_for_fGSEA_all_clusters(fgsea_results, file='fgsea_nes_heatmap.png')
```

## Evaluation signals

- The fgsea_nes_heatmap.png file exists and renders a complete cluster-by-gene-set NES matrix covering all clusters and all 75 gene sets
- Enrichment directions in the NES matrix are consistent with the pre-ranked log2 fold-change ordering of marker genes per cluster
- get_gmtGeneSets(gmt.file) / listAvailableGeneSets(gmt.file) confirms the expected 75 hematopoiesis gene sets were loaded from the GMT file
- Each cluster receives a defensible cell type identity and cell state assignment based on the degree of enrichment of curated gene sets
- fGSEA run is reproducible given a fixed n.seed, with stable NES values across repeated Run_fGSEA_for_multiple_comparisons calls

## Limitations

- SingCellaR requires signature gene sets to perform cell type annotation; for non-hematopoietic systems the user must compile and curate customized gene sets themselves
- Only 75 pre-curated gene sets relevant to hematopoiesis are provided with the protocol
- Plot rotations/appearances can differ from the protocol due to software version and seed number differences
- SingCellaR uses the SingleCellExperiment object and lacks automatic object transformation for interoperability with packages such as Seurat

## Evidence

- [results] A heatmap of GSEA enrichment scores for each gene set across all clusters allows the user to annotate a cell type identity and cell states to each cluster based on the degree of enrichment of the curated gene sets.: [evidence span withheld — anchor status: not_found]
- [methods] 1. Use identifyGSEAPrerankedGenes_for_all_clusters(object, cluster.type, fishers_exact_test, min.expFraction, min.log2FC) on the SingCellaR object to pre-rank differentially expressed genes for every cluster by log2 fold-change.: "identifyGSEAPrerankedGenes_for_all_clusters(object, cluster.type, fishers_exact_test, min.expFraction, min.log2FC)"
- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis.: "we provide 75 gene sets curated from previous studies relevant to hematopoiesis"
- [intro] an integrative analysis tool for analysing single-cell RNA sequencing data: "SingCellaR an integrative analysis tool for analysing single-cell RNA sequencing data"
