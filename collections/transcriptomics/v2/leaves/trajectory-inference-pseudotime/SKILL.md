---
name: trajectory-inference-pseudotime
description: Use when single-cell RNA-seq data has been processed in SingCellaR (Harmony-supervised
  embeddings plus Louvain clusters) and you need pseudotime ordering of immature-to-differentiated
  cells along a lineage such as hematopoietic stem/progenitor to erythroid.
when_to_use_negative:
- Input embeddings and cluster labels have not yet been computed in SingCellaR — trajectory
  transfer presupposes a processed SingCellaR object.
- The goal is de novo dimensionality reduction or integration; this skill reuses existing
  SingCellaR/Harmony embeddings rather than recomputing them.
- The lineage of interest lacks a defined immature root cluster (e.g., no HSC/MPP-equivalent
  population), since order_cells requires an explicit root node.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: 'Source of the processed object: provides supervised Harmony embeddings, Louvain
    clusters, and the normalized UMI matrix transferred into Monocle3'
  repo: https://github.com/supatt-lab/SingCellaR
- name: Monocle3
  role: Learns the principal graph on the transferred embeddings and computes pseudotime
    via learn_graph and order_cells with the cl1 root node
- name: R
  role: Language in which both SingCellaR and Monocle3 run; ggplot-based Monocle3
    plotting renders the trajectory figures
  repo: https://github.com/supatt-lab/SingCellaR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:bedd582270bf691fe41d09fb391f669a426dae2e0749ec746aa310b5ca220636
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# trajectory-inference-pseudotime

## Summary

Infers a cellular differentiation trajectory by transferring SingCellaR-derived Harmony UMAP embeddings and Louvain cluster labels into a Monocle3 cell_data_set, learning a principal graph, and ordering cells in pseudotime from the HSC/MPP (cl1) root to terminal lineages such as the erythroid branch (cl1-cl7-cl12-cl3).

## When to use

Use when single-cell RNA-seq data has been processed in SingCellaR (Harmony-supervised embeddings plus Louvain clusters) and you need pseudotime ordering of immature-to-differentiated cells along a lineage such as hematopoietic stem/progenitor to erythroid.

## When NOT to use

- Input embeddings and cluster labels have not yet been computed in SingCellaR — trajectory transfer presupposes a processed SingCellaR object.
- The goal is de novo dimensionality reduction or integration; this skill reuses existing SingCellaR/Harmony embeddings rather than recomputing them.
- The lineage of interest lacks a defined immature root cluster (e.g., no HSC/MPP-equivalent population), since order_cells requires an explicit root node.

## Inputs

- Processed SingCellaR object with supervised Harmony embeddings (get_umap.result)
- Louvain cluster assignments from SingCellaR (get_clusters)
- Normalized UMI expression matrix from SingCellaR (get_normalized_umi)

## Outputs

- Monocle3 cell_data_set with learned principal trajectory graph
- Per-cell pseudotime values rooted at the cl1 HSC/MPP cluster
- UMMAP trajectory plots colored by pseudotime and branch (Figure 8E-F style)
- Gene-per-pseudotime heatmaps for the erythroid trajectory branch (Figure 8G-I style)

## How to apply

Load the processed SingCellaR object and extract the supervised Harmony embeddings via get_umap.result(object) and Louvain assignments via get_clusters(object). Build a Monocle3 cell_data_set from the normalized expression matrix get_normalized_umi(object), attaching the Harmony embedding as the reduced-dimension space and the cluster labels (cl1, cl7, cl12, cl3, ...) as cell metadata; this manual transfer is necessary because SingCellaR lacks automatic object transformation with other packages. Run Monocle3 learn_graph on the transferred embeddings, then call order_cells designating the cl1 HSC/MPP cluster as the root node so pseudotime increases monotonically toward more differentiated states. Render UMAP plots colored by pseudotime and trajectory branch, and extract pseudotime-ordered expression for erythroid-path genes to draw gene-per-pseudotime heatmaps (reproducing Figure 8E-I).

## Related tools

- **SingCellaR** (Source of the processed object: provides supervised Harmony embeddings, Louvain clusters, and the normalized UMI matrix transferred into Monocle3) — https://github.com/supatt-lab/SingCellaR
- **Monocle3** (Learns the principal graph on the transferred embeddings and computes pseudotime via learn_graph and order_cells with the cl1 root node)
- **R** (Language in which both SingCellaR and Monocle3 run; ggplot-based Monocle3 plotting renders the trajectory figures) — https://github.com/supatt-lab/SingCellaR

## Examples

```
cds <- order_cells(learn_graph(cds_from_singcellar), root_cells = colnames(cds)[clusters(cds) == "cl1"]); plot_cells(cds, color_cells_by = "pseudotime")
```

## Evaluation signals

- The trajectory graph is a single connected principal graph rooted in the HSC/MPP (cl1) cluster.
- Pseudotime increases monotonically from the cl1 root toward the erythroid terminal clusters (cl7, cl12, cl3).
- Trajectory UMAP layout qualitatively matches Figure 8E-F from the protocol.
- Gene-per-pseudotime heatmaps of the erythroid-path gene set reproduce the ordering pattern of Figure 8G-I.
- Cluster labels and reduced-dimension coordinates in the Monocle3 cell_data_set exactly match those returned by get_clusters(object) and get_umap.result(object).

## Limitations

- SingCellaR lacks 'automatic object transformation' to interoperate with other packages (e.g., Seurat), so the embedding/cluster transfer into Monocle3 must be performed manually.
- UMAP/FDG plot rotations can differ from the protocol due to software version and seed number differences.
- Software versions are not stated for the protocol, so exact package versions for Monocle3/SingCellaR may need to be pinned by the practitioner.

## Evidence

- [results] Run Monocle3 learn_graph on the transferred embeddings and call order_cells with the HSC/MPP cluster designated as the root node to compute pseudotime.: [evidence span withheld — anchor status: not_found]
- [methods] Build a Monocle3 cell_data_set from the normalized expression matrix (get_normalized_umi(object)), attaching the Harmony embedding as the reduced-dimension space: [evidence span withheld — anchor status: not_found]
- [discussion] SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat.: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [readme] an integrative analysis tool for analysing single-cell RNA sequencing data: "an integrative analysis tool for analysing single-cell RNA sequencing data"
