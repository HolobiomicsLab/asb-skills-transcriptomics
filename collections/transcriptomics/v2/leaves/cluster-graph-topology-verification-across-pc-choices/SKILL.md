---
name: cluster-graph-topology-verification-across-pc-choices
description: Decision support for whether a trajectory-inference cluster-graph on
  atlas-scale single-cell data is robust to dimensionality-reduction choices, e.g.
  verifying VIA topology on MOCA-scale data across varying ncomps before trusting
  branch and terminal-state structure.
when_to_use_negative:
- When the input data has already been passed through a UMAP or t-SNE embedding that
  will serve as the primary representation — the paper shows UMAP-based approaches
  like Monocle3 are 'highly susceptible to user defined input parameters', so topology
  checks on the embedding do not test graph robustness on PCs
- When the dataset is small enough to inspect directly or has a known simple tree
  topology where PC-choice sensitivity is not a concern
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Trajectory inference method whose PARC-clustered graph topology (terminal
    states, cluster adjacency) is the object being verified across PC choices
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm used inside VIA to build the single-cell graph whose
    cluster-graph nodes must remain globally connected across ncomps values
- name: Monocle3
  role: Negative-control comparator; its UMAP-dependent topology breaks across PC
    choices while VIA's is preserved
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:752351901bf07817e15add3c58ec9cfc164457738f84b31fffa0aec9a355dada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - VIA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Cluster-graph topology verification across PC choices

## Summary

A robustness-verification skill that reruns VIA's PARC-clustered trajectory graph over several numbers of principal components (ncomps) and confirms that the globally connected, multi-branch topology and terminal states persist, as demonstrated on the 1.3-million-cell MOCA mouse organogenesis atlas.

## When to use

Decision support for whether a trajectory-inference cluster-graph on atlas-scale single-cell data is robust to dimensionality-reduction choices, e.g. verifying VIA topology on MOCA-scale data across varying ncomps before trusting branch and terminal-state structure.

## When NOT to use

- When the input data has already been passed through a UMAP or t-SNE embedding that will serve as the primary representation — the paper shows UMAP-based approaches like Monocle3 are 'highly susceptible to user defined input parameters', so topology checks on the embedding do not test graph robustness on PCs
- When the dataset is small enough to inspect directly or has a known simple tree topology where PC-choice sensitivity is not a concern

## Inputs

- AnnData object with 1.3M-cell-scale expression matrix and principal components stored in adata.obsm['X_pca']
- A root/start cell specification (root_user) for VIA
- A set of ncomps values (numbers of principal components) to sweep
- Cell type labels or annotations for identifying fine-grained supergroups and terminal states

## Outputs

- Per-run VIA cluster-graph export (terminal states, cluster-node adjacency, edges)
- Per-PC-choice connected-component count and branch-structure comparison
- Terminal-state correspondence table across PC choices
- Combined topology-persistence check results table

## How to apply

Load the atlas expression matrix as AnnData with PCs in adata.obsm['X_pca'], then run VIA's PARC-based clustering with documented defaults (edgepruning_clustering_resolution ~0.15, too_big_factor 0.4, pseudotime_threshold_TS 30). Export the VIA cluster-graph after each run, then rerun across several ncomps values and compare the graph structure between runs: count connected components, check that fine-grained supergroups remain connected, and confirm adjacency relationships between cluster nodes and terminal-state correspondence. Acceptance is a single connected component with concordant multi-branch adjacency across all tested PC choices. This directly addresses the paper's finding that VIA 'keeps fine-grained supergroups connected and neighborhood relationships preserved across the same range of PCs, unlike UMAP-based Monocle3', which is highly susceptible to user-defined input parameters.

## Related tools

- **VIA** (Trajectory inference method whose PARC-clustered graph topology (terminal states, cluster adjacency) is the object being verified across PC choices) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm used inside VIA to build the single-cell graph whose cluster-graph nodes must remain globally connected across ncomps values)
- **Monocle3** (Negative-control comparator; its UMAP-dependent topology breaks across PC choices while VIA's is preserved)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset=''); # rerun with ncomps in {20, 30, 50} and export the cluster-graph after each run to compare connectivity and terminal states
```

## Evaluation signals

- After each ncomps rerun, the VIA cluster-graph has exactly one connected component (globally connected topology)
- Fine-grained supergroups stay connected — no supergroup is split into disconnected nodes when PC count varies
- Neighborhood/adjacency relationships between cluster nodes (e.g. three-branch structure) are concordant across all tested PC choices
- Terminal states identified at pseudotime_threshold_TS = 30 correspond one-to-one across runs with different ncomps
- Runtime remains on the order of tens of minutes (~40 min on 1.3M cells), confirming the configuration is still the scalable VIA path rather than a degenerate setting

## Limitations

- The paper does not state software version numbers for VIA or the compared methods, so topology-persistence results may not be reproducible across versions
- MOCA is referenced without an accession or repository identifier in the paper, so obtaining the exact validation dataset requires tracing the GEO deposit externally
- Specific parameter ranges used in the paper's stress tests are not reported in the discussion; only default key parameters (jac_std_global ~0.15, too_big_factor 0.4, pseudotime_threshold_TS 30) are documented in methods
- Methods requiring UMAP or t-SNE before parsing large atlases are 'highly susceptible to user defined input parameters', so topology conclusions should be drawn from the PCA-based graph, not the 2D embedding

## Evidence

- [discussion] VIA keeps fine-grained supergroups connected and neighborhood relationships preserved across the same range of PCs, unlike UMAP-based Monocle3.: "We also demonstrated on the 1.3 million MOCA dataset that VIA is highly scalable with a runtime of ~40 min"
- [abstract] VIA resolves the atlas without losing global connectivity.: "VIA robustly and efficiently unravels the fine-grained sub-trajectories in a 1.3-million-cell transcriptomic mouse atlas without losing the global connectivity"
- [discussion] UMAP-based methods are parameter-sensitive on MOCA.: "We showed that methods which require UMAP (or t-SNE) before parsing MOCA are highly susceptible to user defined input parameters"
- [methods] Key VIA parameters for the graph construction and terminal-state detection are documented.: "global level  graph pruning for PARC clustering stage. Key tuning parameter."
- [intro] VIA scales across cell counts without destructive reduction steps.: "VIA is highly scalable with respect to number of cells (102 to >106 cells) and features, without requiring extensive dimensionality reduction or subsampling which compromise global information"
- [methods] VIA input format is AnnData with PCA in obsm.: "adata = sc.AnnData(data_genes)"
