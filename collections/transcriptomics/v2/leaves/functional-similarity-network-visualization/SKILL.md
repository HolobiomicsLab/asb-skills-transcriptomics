---
name: functional-similarity-network-visualization
description: Use when a plant single-cell atlas contains clusters with unknown marker
  genes or unclear identity and you want to infer their cell type from GO-term enrichment
  profiles via topic modeling and a functional similarity network.
when_to_use_negative:
- Input is non-plant scRNA-seq data without a plant knowledgebase (e.g., TAIR/GO annotations)
  — scPlant's GSEA pipeline is built around plant genome resources
- Clusters already have confident marker-based annotations and the goal is marker
  discovery or trajectory inference rather than functional annotation
- Input is bulk transcriptomic data — scPlant targets single-cell atlases and uses
  Seurat/CellFunTopic single-cell objects
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: scPlant
  role: Provides RunGSEA_plant for GO-based gene set enrichment scoring of clusters
    (by='GO', GeneIDtype='TAIR', minpct=0.25)
  repo: https://github.com/compbioNJU/scPlant
- name: CellFunTopic
  role: Performs LDA topic modeling (runLDA, k=23) and visualizes GSEA results and
    the functional similarity network (pathway_unique_shared, hierarchyplot_tree)
  repo: https://github.com/compbioNJU/CellFunTopic
- name: Seurat
  role: Provides the Seurat object container and basic single-cell processing that
    scPlant integrates with
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:4193fa3aa0aacf3909e8e4aa1108e8052861618b1edaa4a0e119a8bfe6645830
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
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# functional-similarity-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A scPlant/CellFunTopic-based workflow that annotates cell clusters functionally by combining GO-based GSEA scoring (RunGSEA_plant) with LDA topic modeling (runLDA) to build a cluster-level functional similarity network. Applied to the Arabidopsis thaliana root atlas (Zhang et al., 2019), it places the unannotated cluster 18 adjacent to cortex clusters.

## When to use

Use when a plant single-cell atlas contains clusters with unknown marker genes or unclear identity and you want to infer their cell type from GO-term enrichment profiles via topic modeling and a functional similarity network.

## When NOT to use

- Input is non-plant scRNA-seq data without a plant knowledgebase (e.g., TAIR/GO annotations) — scPlant's GSEA pipeline is built around plant genome resources
- Clusters already have confident marker-based annotations and the goal is marker discovery or trajectory inference rather than functional annotation
- Input is bulk transcriptomic data — scPlant targets single-cell atlases and uses Seurat/CellFunTopic single-cell objects

## Inputs

- Annotated Seurat object of a plant scRNA-seq dataset (e.g., Arabidopsis thaliana root, Zhang et al., 2019)
- TAIR gene IDs compatible with GO-based enrichment (TAIR10 genome annotation)
- Number of clusters (used to set LDA topic count k, e.g., k=23)

## Outputs

- Per-cluster GO pathway enrichment signatures (GSEA scores in the Seurat object)
- LDA beta and gamma matrices (topic-pathway and cluster-topic, extracted via tidytext::tidy)
- Cluster-level functional similarity network / hierarchy tree (e.g., hierarchyplot_tree with cluster_cutree_k=14, pathway_cutree_k=14)
- Functional annotation or proximity assignment for unannotated clusters (e.g., cluster 18 near cortex clusters)

## How to apply

Load the annotated Arabidopsis thaliana root Seurat object (Zhang et al., 2019 data from the scPlant figshare deposit, https://figshare.com/articles/dataset/Auxiliary_data_and_codes_for_scPlant/23498402) into R with scPlant and Seurat. Score each cluster with scPlant's RunGSEA_plant using by='GO', GeneIDtype='TAIR', and minpct=0.25 so each cluster receives enriched GO pathway signatures. Visualize the GSEA result with CellFunTopic functions such as pathway_unique_shared and hierarchyplot_tree to inspect pathways shared across or exclusive to cell types. Fit an LDA topic model on the cluster-by-pathway matrix with CellFunTopic's runLDA using k=23 topics (matching the number of clusters), extracting beta and gamma matrices via tidytext::tidy. Compute cluster-level functional similarity from the topic/pathway profiles and draw the similarity network (hierarchyplot_tree, cluster_cutree_k=14, pathway_cutree_k=14); the rationale is that topics summarize shared GO-term usage so clusters with similar functions sit near each other in the network.

## Related tools

- **scPlant** (Provides RunGSEA_plant for GO-based gene set enrichment scoring of clusters (by='GO', GeneIDtype='TAIR', minpct=0.25)) — https://github.com/compbioNJU/scPlant
- **CellFunTopic** (Performs LDA topic modeling (runLDA, k=23) and visualizes GSEA results and the functional similarity network (pathway_unique_shared, hierarchyplot_tree)) — https://github.com/compbioNJU/CellFunTopic
- **Seurat** (Provides the Seurat object container and basic single-cell processing that scPlant integrates with)

## Examples

```
SeuratObj <- RunGSEA_plant(SeuratObj, by = 'GO', GeneIDtype = 'TAIR', minpct = 0.25); SeuratObj <- runLDA(SeuratObj, k = 23)
```

## Evaluation signals

- The LDA model must yield 23 topics over the GO-enriched clusters (k matching the number of clusters)
- The resulting functional-similarity network must place unannotated cluster 18 adjacent to cortex clusters, reproducing Figure 3F of the paper
- RunGSEA_plant returns enriched GO pathway signatures for every cluster with minpct=0.25 and TAIR gene IDs
- Beta and gamma matrices are extractable via tidytext::tidy from the fitted CellFunTopic LDA model
- Clustering the similarity network (cluster_cutree_k=14, pathway_cutree_k=14) yields coherent pathway/cluster groups consistent with known root cell types

## Limitations

- Success criterion is qualitative: cluster 18's cortex proximity is judged visually from the similarity network rather than by a quantitative threshold
- Requires prior cluster number to set the LDA topic count (k=23 topics for 23 clusters); re-clustering requires re-tuning k
- Depends on plant-specific resources (TAIR10 genome, GO annotations); not directly transferable to non-plant species

## Evidence

- [readme] CellFunTopic purpose per its README: [evidence span withheld — anchor status: external_doc]
