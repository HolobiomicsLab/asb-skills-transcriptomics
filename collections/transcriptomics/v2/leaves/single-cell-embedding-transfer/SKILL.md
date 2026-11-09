---
name: single-cell-embedding-transfer
description: Use when a processed SingCellaR object already contains Harmony/UMAP
  embeddings and Louvain cluster labels, and trajectory or pseudotime ordering (e.g.
  HSC/MPP to erythroid differentiation) is needed in Monocle3.
when_to_use_negative:
- The input is raw count matrices or an unprocessed SingCellaR object that has not
  yet been normalized (normalize_UMIs) and clustered — run the upstream SingCellaR
  QC and clustering workflow first
- The downstream tool already provides its own embeddings and you do not need to preserve
  the SingCellaR-derived representation (Monocle3 can compute its own UMAP)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Source of the Harmony embeddings, Louvain cluster assignments, and normalized
    UMI matrix that are exported into the downstream trajectory tool
  repo: github.com/supatt-lab/SingCellaR
- name: Monocle3
  role: Receives the transferred embeddings and cluster labels; performs learn_graph
    and order_cells to compute pseudotime from the designated root node
- name: R
  role: Language in which both SingCellaR and Monocle3 run; the transfer is done by
    rebuilding the cell_data_set in R from SingCellaR accessors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:077b6a4abcd142d0a01ea9ce1558cc0fafed269fbd501f0468a2213d425ccac3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle3
  - R
  - SingCellaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-embedding-transfer

## Summary

Transfers dimensionality-reduction embeddings and cluster assignments computed in a SingCellaR object into a downstream tool (Monocle3) by rebuilding a cell_data_set from the normalized UMI matrix, enabling trajectory/pseudotime analysis on the SingCellaR-derived representation rather than Monocle3's default UMAP.

## When to use

Use when a processed SingCellaR object already contains Harmony/UMAP embeddings and Louvain cluster labels, and trajectory or pseudotime ordering (e.g. HSC/MPP to erythroid differentiation) is needed in Monocle3.

## When NOT to use

- The input is raw count matrices or an unprocessed SingCellaR object that has not yet been normalized (normalize_UMIs) and clustered — run the upstream SingCellaR QC and clustering workflow first
- The downstream tool already provides its own embeddings and you do not need to preserve the SingCellaR-derived representation (Monocle3 can compute its own UMAP)

## Inputs

- Processed SingCellaR object with supervised Harmony embeddings (get_umap.result)
- Louvain cluster assignments from the SingCellaR object (get_clusters)
- Normalized UMI expression matrix (get_normalized_umi)
- Root cluster designation (e.g. HSC/MPP cluster cl1) and target branch/gene set (e.g. erythroid path cl1-cl7-cl12-cl3)

## Outputs

- Monocle3 cell_data_set with SingCellaR embeddings and cluster metadata
- Trajectory principal graph and per-cell pseudotime values rooted at the HSC/MPP cluster
- UMAP trajectory plots colored by pseudotime and branch (Figure 8E-F analogues)
- Gene-per-pseudotime heatmaps for the erythroid trajectory branch (Figure 8G-I analogues)

## How to apply

Extract the supervised Harmony embeddings from the SingCellaR object with get_umap.result(object) and the Louvain cluster assignments with get_clusters(object), then build a Monocle3 cell_data_set from the normalized expression matrix get_normalized_umi(object), attaching the Harmony embedding as the reduced-dimension space and cluster labels as cell metadata. Run Monocle3 learn_graph on the transferred embeddings, then call order_cells designating the HSC/MPP cluster (cl1) as the root node to compute pseudotime. Render UMAP plots colored by pseudotime and trajectory branch (e.g. the erythroid path cl1-cl7-cl12-cl3) with ggplot-based Monocle3 plotting, and extract pseudotime-ordered expression for branch-specific gene sets to draw gene-per-pseudotime heatmaps. The rationale is that SingCellaR uses the SingleCellExperiment object as its data container and lacks automatic object transformation for other packages, so embeddings must be manually exported and attached to the downstream object.

## Related tools

- **SingCellaR** (Source of the Harmony embeddings, Louvain cluster assignments, and normalized UMI matrix that are exported into the downstream trajectory tool) — github.com/supatt-lab/SingCellaR
- **Monocle3** (Receives the transferred embeddings and cluster labels; performs learn_graph and order_cells to compute pseudotime from the designated root node)
- **R** (Language in which both SingCellaR and Monocle3 run; the transfer is done by rebuilding the cell_data_set in R from SingCellaR accessors)

## Evaluation signals

- The trajectory graph is a single connected principal graph rooted in the HSC/MPP cluster (cl1)
- Pseudotime increases monotonically from the HSC/MPP root toward the erythroid terminal clusters
- The UMAP layout of the Monocle3 plot qualitatively matches the SingCellaR embeddings and the Figure 8E-I panel layout
- Cell metadata cluster labels in the cell_data_set match get_clusters(object) counts from the SingCellaR object
- Gene-per-pseudotime heatmaps for the erythroid-path gene set show ordered expression transitions along cl1-cl7-cl12-cl3

## Limitations

- SingCellaR lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, so embeddings/clusters must be exported manually rather than via object conversion
- UMAP/FDG plot rotations may differ from the protocol figures due to software version and seed number differences
- The protocol was demonstrated on human hematopoietic stem and progenitor cells; root cluster and branch definitions (cl1 HSC/MPP, erythroid cl1-cl7-cl12-cl3) must be re-derived for other systems

## Evidence

- [discussion] SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R.: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [discussion] SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat.: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [methods] normalize_UMIs call: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [readme] README description of SingCellaR: "an integrative analysis tool for analysing single-cell RNA sequencing data"
