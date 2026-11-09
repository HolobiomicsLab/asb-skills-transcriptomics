---
name: cluster-marker-gene-ranking
description: Use when scRNA-seq clusters (e.g. integrated HSPC clusters from SingCellaR)
  need systematic, gene-set-based cell type annotation rather than marker-gene eyeballing,
  and a curated GMT of lineage signatures is available.
when_to_use_negative:
- No curated signature gene sets exist for the system of interest — the article notes
  the user would have to compile and curate customized gene sets, so annotation cannot
  proceed reliably
- Clusters have not yet been defined (clustering not performed), so there is no cluster.type
  to rank markers against
- The goal is marker gene selection for a two-group comparison rather than gene-set-based
  cluster annotation (use identifyDifferentialGenes instead)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R package providing identifyGSEAPrerankedGenes_for_all_clusters, Run_fGSEA_for_multiple_comparisons,
    and plot_heatmap_for_fGSEA_all_clusters for marker ranking and gene-set-based
    cluster annotation
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment in which the SingCellaR annotation workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:7d38407fb5b496e600830fab58e7cc6e7562f7887f61d13f430e98bb369d4a69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cluster-marker-gene-ranking

## Summary

Ranks differentially expressed genes per cluster by log2 fold-change and uses pre-ranked GSEA (fGSEA) against curated signature gene sets to objectively assign cell type identities and cell states to integrated scRNA-seq clusters, visualized as an NES heatmap.

## When to use

Use when scRNA-seq clusters (e.g. integrated HSPC clusters from SingCellaR) need systematic, gene-set-based cell type annotation rather than marker-gene eyeballing, and a curated GMT of lineage signatures is available.

## When NOT to use

- No curated signature gene sets exist for the system of interest — the article notes the user would have to compile and curate customized gene sets, so annotation cannot proceed reliably
- Clusters have not yet been defined (clustering not performed), so there is no cluster.type to rank markers against
- The goal is marker gene selection for a two-group comparison rather than gene-set-based cluster annotation (use identifyDifferentialGenes instead)

## Inputs

- SingCellaR object with clustered, integrated scRNA-seq data (e.g. human HSPC clusters)
- cluster.type definition for the object
- Curated GMT file of signature gene sets (75 hematopoiesis gene sets)
- fGSEA parameters: minSize, maxSize, eps, nPermSimple, n.seed

## Outputs

- Per-cluster pre-ranked gene tables ordered by log2FC
- NES and enrichment p-value matrix (clusters x gene sets)
- fgsea_nes_heatmap.png — cluster-by-gene-set NES heatmap
- Assigned cell type identity and cell state per cluster

## How to apply

On the SingCellaR object, run identifyGSEAPrerankedGenes_for_all_clusters(object, cluster.type, fishers_exact_test, min.expFraction, min.log2FC) to pre-rank each cluster's differentially expressed genes by log2 fold-change. Verify the gene-set universe with get_gmtGeneSets(gmt.file) / listAvailableGeneSets(gmt.file) against the curated 75 hematopoiesis gene set GMT file. Pass the list of pre-ranked gene tables to Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed) to compute normalized enrichment scores (NES) and enrichment p-values for every cluster against every gene set. Finally render the NES matrix across all clusters with plot_heatmap_for_fGSEA_all_clusters and save as fgsea_nes_heatmap.png; annotate each cluster's identity and cell state from the degree and direction of enrichment of the curated gene sets.

## Related tools

- **SingCellaR** (R package providing identifyGSEAPrerankedGenes_for_all_clusters, Run_fGSEA_for_multiple_comparisons, and plot_heatmap_for_fGSEA_all_clusters for marker ranking and gene-set-based cluster annotation) — https://github.com/supatt-lab/SingCellaR
- **R** (Runtime environment in which the SingCellaR annotation workflow is executed)

## Examples

```
Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed)
```

## Evaluation signals

- fgsea_nes_heatmap.png exists and renders a complete cluster-by-gene-set NES matrix covering all clusters and all 75 gene sets
- Enrichment directions in the NES heatmap are consistent with the pre-ranked log2FC ordering of genes in each cluster
- get_gmtGeneSets(gmt.file) / listAvailableGeneSets(gmt.file) returns the expected gene-set universe (75 hematopoiesis sets) before running fGSEA
- Each cluster receives a reproducible annotation (identity + cell state) supported by the degree of enrichment of specific curated gene sets
- Setting n.seed yields stable NES values across reruns (modulo documented seed/plot-rotation variability)

## Limitations

- SingCellaR requires signature gene sets for annotation; only 75 pre-curated hematopoiesis gene sets are provided — other systems require user-curated GMT files
- Plot rotations and enrichment outputs can differ slightly due to software version and seed number differences
- SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat, complicating export of rankings
- No software versions are stated for the protocol, which may affect reproducibility

## Evidence

- [discussion] GSEA enrichment heatmap for cell type annotation: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] Gene set curation limitation: "the user would have to compile and curate customized gene sets for the relevant system of interest. In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [full_text] fGSEA workflow definition from source card: [evidence span withheld — anchor status: not_found]
- [readme] Tool documentation availability: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
