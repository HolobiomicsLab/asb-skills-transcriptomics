---
name: hematopoiesis-gene-set-interpretation
description: Use when clusters from an integrated hematopoietic (e.g. HSPC) scRNA-seq
  dataset need objective cell-type labels, and a curated GMT of hematopoiesis signature
  gene sets is available.
when_to_use_negative:
- The tissue or lineage is not hematopoietic — the protocol only provides 75 gene
  sets curated from previous studies relevant to hematopoiesis, so users must compile
  and curate customized gene sets for other systems
- You need fully automatic annotation without curated signature sets — SingCellaR
  requires signature gene sets to perform the cell type annotation analysis
- Input is raw unnormalized UMI counts without clusters or differential expression
  computed — pre-ranked genes per cluster must exist first
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Provides identifyGSEAPrerankedGenes_for_all_clusters, Run_fGSEA_for_multiple_comparisons,
    plot_heatmap_for_fGSEA_all_clusters, and GMT inspection functions used to annotate
    clusters from NES enrichment
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment for the SingCellaR analysis package and fgSEA workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:92f226d2bcdf8e6b6a7edc7efe993e88f2ba002d195839e29a22b805643a95c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hematopoiesis-gene-set-interpretation

## Summary

Systematically annotate cell type identities and cell states for integrated scRNA-seq clusters by running fgSEA with 75 curated hematopoiesis gene sets in SingCellaR and interpreting a cluster-by-gene-set NES heatmap.

## When to use

Use when clusters from an integrated hematopoietic (e.g. HSPC) scRNA-seq dataset need objective cell-type labels, and a curated GMT of hematopoiesis signature gene sets is available.

## When NOT to use

- The tissue or lineage is not hematopoietic — the protocol only provides 75 gene sets curated from previous studies relevant to hematopoiesis, so users must compile and curate customized gene sets for other systems
- You need fully automatic annotation without curated signature sets — SingCellaR requires signature gene sets to perform the cell type annotation analysis
- Input is raw unnormalized UMI counts without clusters or differential expression computed — pre-ranked genes per cluster must exist first

## Inputs

- SingCellaR object with integrated HSPC clusters and cluster.type assignments
- Curated GMT file of 75 hematopoiesis signature gene sets
- Per-cluster gene tables pre-ranked by log2 fold-change

## Outputs

- fgsea NES and enrichment p-values per cluster per gene set
- fgsea_nes_heatmap.png: cluster-by-gene-set NES heatmap used to assign cell type identity and cell states to each cluster

## How to apply

First use identifyGSEAPrerankedGenes_for_all_clusters on the SingCellaR object to pre-rank differentially expressed genes per cluster by log2 fold-change (with fishers_exact_test, min.expFraction and min.log2FC filters). Verify the gene-set universe against the curated 75 hematopoiesis GMT file using get_gmtGeneSets / listAvailableGeneSets. Then run Run_fGSEA_for_multiple_comparisons on the list of pre-ranked gene tables to compute NES and enrichment p-values per cluster per gene set, and render the NES matrix with plot_heatmap_for_fGSEA_all_clusters (saved as fgsea_nes_heatmap.png). The rationale is that the degree and direction of enrichment of lineage/state gene sets provides an objective, gene-set-level basis for naming each cluster rather than ad-hoc inspection of marker genes.

## Related tools

- **SingCellaR** (Provides identifyGSEAPrerankedGenes_for_all_clusters, Run_fGSEA_for_multiple_comparisons, plot_heatmap_for_fGSEA_all_clusters, and GMT inspection functions used to annotate clusters from NES enrichment) — https://github.com/supatt-lab/SingCellaR
- **R** (Runtime environment for the SingCellaR analysis package and fgSEA workflow)

## Examples

```
Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed)
```

## Evaluation signals

- fgsea_nes_heatmap.png exists and renders a complete cluster-by-gene-set NES matrix covering all clusters and the 75 gene sets
- Enrichment directions in the NES matrix are consistent with the pre-ranked log2FC gene ordering for each cluster
- get_gmtGeneSets / listAvailableGeneSets confirms the GMT parses to the expected 75 hematopoiesis gene sets
- Each cluster receives a cell type identity and cell state call justified by its most enriched gene set(s) in the heatmap

## Limitations

- SingCellaR requires signature gene sets to perform the cell type annotation analysis; only 75 pre-curated hematopoiesis gene sets are provided, so other systems require user-curated gene sets
- Heatmap appearance can vary with software version and seed number differences
- SingCellaR lacks automatic object transformation for interoperability with other packages such as Seurat

## Evidence

- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] 75 curated hematopoiesis gene sets: "In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [methods] GSEA heatmap-based annotation workflow: [evidence span withheld — anchor status: not_found]
- [readme] SingCellaR description in README: "an integrative analysis tool for analysing single-cell RNA sequencing data"
