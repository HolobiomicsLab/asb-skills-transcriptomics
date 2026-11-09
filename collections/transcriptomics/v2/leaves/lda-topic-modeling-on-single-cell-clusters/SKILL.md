---
name: lda-topic-modeling-on-single-cell-clusters
description: Use when single-cell clusters contain unannotated populations with unknown
  marker genes and you need functional annotation from GO enrichment signatures plus
  topic-model-based similarity, especially for plant scRNA-seq.
when_to_use_negative:
- All clusters are already confidently annotated by marker genes or reference-based
  methods, so functional topic modeling adds no identity information
- The input is not a cluster-level pathway/functional signature matrix (e.g., raw
  counts or a feature table) — first run GSEA scoring with RunGSEA_plant
- The organism lacks a compatible plant GO knowledgebase mapping for the gene ID type
  (e.g., non-TAIR identifiers without conversion)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPlant
  role: Provides RunGSEA_plant to build the GO-based cluster-by-pathway enrichment
    matrix that the LDA model is fit on
  repo: https://github.com/compbioNJU/scPlant
- name: CellFunTopic
  role: Performs LDA topic modeling (runLDA) and functional-similarity visualization
    (pathway_unique_shared, hierarchyplot_tree) on the GSEA result
  repo: https://github.com/compbioNJU/CellFunTopic
- name: Seurat
  role: Holds the annotated scRNA-seq object that RunGSEA_plant and runLDA operate
    on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:d52f3fac6c594184dc2de69d647511e4dde23886aa5927e4541805c7aa453d1b
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lda-topic-modeling-on-single-cell-clusters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit an LDA topic model with CellFunTopic's runLDA on a cluster-by-GO-pathway matrix (built from scPlant's RunGSEA_plant output) to derive functional profiles of cell clusters and identify the identity of unannotated clusters via functional similarity networks, as demonstrated on Zhang et al. (2019) Arabidopsis root data where cluster 18 grouped with cortex clusters.

## When to use

Use when single-cell clusters contain unannotated populations with unknown marker genes and you need functional annotation from GO enrichment signatures plus topic-model-based similarity, especially for plant scRNA-seq.

## When NOT to use

- All clusters are already confidently annotated by marker genes or reference-based methods, so functional topic modeling adds no identity information
- The input is not a cluster-level pathway/functional signature matrix (e.g., raw counts or a feature table) — first run GSEA scoring with RunGSEA_plant
- The organism lacks a compatible plant GO knowledgebase mapping for the gene ID type (e.g., non-TAIR identifiers without conversion)

## Inputs

- Annotated Seurat object of plant scRNA-seq clusters (e.g., Arabidopsis thaliana root, Zhang et al. 2019)
- GO pathway/gene set definitions compatible with TAIR gene IDs

## Outputs

- Cluster-by-GO-pathway enrichment matrix (RunGSEA_plant output stored in the Seurat object)
- LDA topic model with beta and gamma matrices extracted via tidytext::tidy
- Functional similarity network/hierarchy plot linking clusters, e.g., cluster 18 adjacent to cortex clusters (paper Figure 3F)

## How to apply

Load the annotated Seurat object (e.g., the Zhang et al. 2019 Arabidopsis thaliana root data from the scPlant figshare deposit) into R with scPlant and Seurat. Score each cluster with GO-based gene set enrichment via scPlant's RunGSEA_plant using by='GO', GeneIDtype='TAIR', and minpct=0.25. Optionally visualize shared/exclusive pathways with CellFunTopic functions such as pathway_unique_shared and hierarchyplot_tree. Then fit an LDA topic model on the cluster-by-pathway matrix with CellFunTopic's runLDA, setting k to the number of clusters (k=23 for the 23 Arabidopsis root clusters) and extracting the beta and gamma matrices with tidytext::tidy. Finally, compute cluster-level functional similarity from the topic/pathway profiles and draw the similarity network (e.g., hierarchyplot_tree with pathway_cutree_k=14 and cluster_cutree_k=14); an unannotated cluster placed adjacent to a known cell type supports assigning that identity.

## Related tools

- **scPlant** (Provides RunGSEA_plant to build the GO-based cluster-by-pathway enrichment matrix that the LDA model is fit on) — https://github.com/compbioNJU/scPlant
- **CellFunTopic** (Performs LDA topic modeling (runLDA) and functional-similarity visualization (pathway_unique_shared, hierarchyplot_tree) on the GSEA result) — https://github.com/compbioNJU/CellFunTopic
- **Seurat** (Holds the annotated scRNA-seq object that RunGSEA_plant and runLDA operate on)

## Examples

```
SeuratObj <- RunGSEA_plant(SeuratObj, by = 'GO', GeneIDtype = 'TAIR', minpct = 0.25); SeuratObj <- runLDA(SeuratObj, k = 23)
```

## Evaluation signals

- The LDA model runs with k equal to the number of clusters (k=23 for the Arabidopsis root dataset) and yields 23 topics over the GO-enriched clusters
- beta and gamma matrices are extractable via tidytext::tidy from the fitted model
- The functional similarity network qualitatively reproduces the paper's Figure 3F, placing unannotated cluster 18 adjacent to cortex clusters
- GSEA scoring used the documented parameters (by='GO', GeneIDtype='TAIR', minpct=0.25) so each cluster receives enriched GO pathway signatures
- Dendrogram/cutree steps run with the documented settings (cluster_cutree_k=14, pathway_cutree_k=14) without errors

## Limitations

- Validation of cluster identity is qualitative: success depends on the similarity network reproducing the expected adjacency (cluster 18 with cortex), not on a quantitative accuracy metric
- Requires a plant GO knowledgebase and TAIR-compatible gene IDs; the paper notes existing guidelines 'are not immediately applicable to the plant community owing to lack of a plant knowledgebase'
- The number of LDA topics must match the number of clusters (k=23), which must be set manually per dataset

## Evidence

- [readme] CellFunTopic README describing its purpose: [evidence span withheld — anchor status: external_doc]
