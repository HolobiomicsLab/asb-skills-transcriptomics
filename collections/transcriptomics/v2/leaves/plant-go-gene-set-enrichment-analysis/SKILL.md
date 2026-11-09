---
name: plant-go-gene-set-enrichment-analysis
description: Use when clusters from a plant scRNA-seq atlas contain cells with unknown
  marker genes and you need GO-based functional annotation, or when you must test
  whether an unannotated cluster resembles a known cell type via functional similarity.
when_to_use_negative:
- Input is raw counts that have not passed scPlant basic processing (QCfun, RunSeurat)
  — GSEA scores depend on prior normalization and clustering
- Non-plant species without a plant knowledgebase/GO gene sets keyed to the relevant
  gene ID type (e.g. not TAIR)
- When you need marker-based cell-type annotation rather than pathway-level functional
  annotation — SingleR, Garnett, or scCATCH in scPlant would be the appropriate module
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides RunGSEA_plant to compute GO-based gene set enrichment per cluster
    using TAIR gene IDs
  repo: https://github.com/compbioNJU/scPlant
- name: CellFunTopic
  role: Visualizes GSEA results and performs LDA topic modeling on the cluster-by-pathway
    matrix (runLDA, pathway_unique_shared, hierarchyplot_tree)
  repo: https://github.com/compbioNJU/CellFunTopic
- name: Seurat
  role: Provides the Seurat object data structure that scPlant's RunGSEA_plant operates
    on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:108627b14e96f25066fb7c9597a546574f02131764cf9730972186a53d36a9f8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - CellFunTopic
  - Seurat
  - scPlant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# plant-go-gene-set-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores plant single-cell clusters with GO-derived gene set enrichment (scPlant's RunGSEA_plant on TAIR gene IDs) and then interprets the cluster-by-pathway matrix with CellFunTopic visualization and LDA topic modeling to assign functional identity to cell types, e.g. linking unknown cluster 18 to cortex clusters in the Arabidopsis thaliana root atlas (Zhang et al., 2019).

## When to use

Use when clusters from a plant scRNA-seq atlas contain cells with unknown marker genes and you need GO-based functional annotation, or when you must test whether an unannotated cluster resembles a known cell type via functional similarity.

## When NOT to use

- Input is raw counts that have not passed scPlant basic processing (QCfun, RunSeurat) — GSEA scores depend on prior normalization and clustering
- Non-plant species without a plant knowledgebase/GO gene sets keyed to the relevant gene ID type (e.g. not TAIR)
- When you need marker-based cell-type annotation rather than pathway-level functional annotation — SingleR, Garnett, or scCATCH in scPlant would be the appropriate module

## Inputs

- Annotated plant Seurat object (e.g. Arabidopsis thaliana root scRNA-seq, Zhang et al. 2019, from the scPlant figshare deposit 23498402)
- GO gene sets referenced by TAIR gene IDs
- Number of clusters (used as LDA topic count k)

## Outputs

- Per-cluster enriched GO pathway signatures
- CellFunTopic visualizations (pathway_unique_shared, hierarchyplot_tree)
- LDA beta (topic-pathway) and gamma (cluster-topic) matrices via tidytext::tidy
- Cluster-level functional similarity network / hierarchy clustering placing unknown clusters relative to annotated cell types

## How to apply

Load the annotated plant Seurat object (e.g. the Arabidopsis thaliana root data from the Zhang et al. 2019 deposit on the scPlant figshare page) and run scPlant's RunGSEA_plant with by='GO', GeneIDtype='TAIR', and minpct=0.25 so each cluster receives enriched GO pathway signatures. Inspect shared versus cell-type-exclusive pathways with CellFunTopic functions such as pathway_unique_shared and hierarchyplot_tree. Fit an LDA topic model on the cluster-by-pathway matrix with CellFunTopic's runLDA, setting k equal to the number of clusters (k=23 for the 23 Arabidopsis root clusters), and extract beta/gamma matrices via tidytext::tidy. Compute cluster-level functional similarity from the topic/pathway profiles and draw the similarity network; a correct run places the unannotated cluster (18) adjacent to cortex clusters, reproducing the paper's Figure 3F.

## Related tools

- **scPlant** (Provides RunGSEA_plant to compute GO-based gene set enrichment per cluster using TAIR gene IDs) — https://github.com/compbioNJU/scPlant
- **CellFunTopic** (Visualizes GSEA results and performs LDA topic modeling on the cluster-by-pathway matrix (runLDA, pathway_unique_shared, hierarchyplot_tree)) — https://github.com/compbioNJU/CellFunTopic
- **Seurat** (Provides the Seurat object data structure that scPlant's RunGSEA_plant operates on)

## Examples

```
SeuratObj <- RunGSEA_plant(SeuratObj, by = 'GO', GeneIDtype = 'TAIR', minpct = 0.25); SeuratObj <- runLDA(SeuratObj, k = 23)
```

## Evaluation signals

- RunGSEA_plant completes with by='GO', GeneIDtype='TAIR', minpct=0.25 and every cluster receives at least one enriched GO signature
- runLDA yields exactly k=23 topics, matching the number of Arabidopsis root clusters
- The functional similarity network places the unannotated cluster 18 adjacent/connected to cortex clusters, as in Figure 3F
- Hierarchy plots (pathway_cutree_k=14, cluster_cutree_k=14) resolve sensible pathway and cluster groupings
- Beta and gamma matrices are extractable via tidytext::tidy and align with the cluster and pathway labels of the input object

## Limitations

- Success is qualitative — the validation criterion is that the similarity network places cluster 18 next to cortex clusters, so no quantitative cutoff replaces visual confirmation against the reference figure
- Requires a plant GO knowledgebase keyed to the organism's gene IDs (TAIR for Arabidopsis); other organisms must supply genome/GFF from EnsemblPlants or Phytozome
- k must match the number of clusters; a different clustering resolution in RunSeurat changes the required topic count

## Evidence
