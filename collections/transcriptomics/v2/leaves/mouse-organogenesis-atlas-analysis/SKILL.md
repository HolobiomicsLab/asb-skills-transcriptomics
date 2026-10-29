---
name: mouse-organogenesis-atlas-analysis
description: Use when running trajectory inference on a ~1.3-million-cell single-cell
  atlas (e.g., MOCA) and needing to confirm scalable runtime (~40 min) plus globally
  connected, multi-branch topology robustness across PC counts.
when_to_use_negative:
- Input is a small (~10^2-10^3 cell) simulated or toy dataset — dyntoy/dyngen simulations
  are for prototyping TI methods, not for this million-cell scalability benchmark
- The goal is topology discovery on data where UMAP/t-SNE pre-embedding suffices —
  the article notes methods requiring UMAP (or t-SNE) before parsing MOCA are highly
  susceptible to user-defined input parameters
- Data are disconnected trajectories needing multiple roots — use via_wrapper_disconnected
  instead of the standard atlas-scale connected workflow
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: Primary scalable trajectory inference method; lazy-teleporting random walks
    over a PARC-clustered graph to compute pseudotime and lineages on the MOCA atlas
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm used inside VIA to build the single-cell cluster-graph
    nodes prior to trajectory computation
- name: Palantir
  role: Baseline TI method timed against VIA's ~40 min runtime on MOCA (>4 h)
- name: STREAM
  role: Baseline TI method timed against VIA on MOCA (>6.5 h)
- name: Monocle3
  role: UMAP-dependent comparator whose fine-grained supergroups lose connectivity
    on MOCA across PC choices
- name: UMAP
  role: Embedding used for single-cell-level visualization of the VIA trajectory,
    not for the TI computation itself
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:7c15d9429be588e37500e4adf9bf2d4cb61d9c176a871c42fa2474a0643acb41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - UMAP
  - VIA
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mouse-organogenesis-atlas-analysis

## Summary

Benchmark and reproduce VIA's million-cell-scale trajectory inference on the 1.3-million-cell MOCA mouse organogenesis atlas, measuring runtime against PAGA, Palantir, STREAM and Monocle3 while verifying that the cluster-graph preserves global connectivity across choices of input principal components.

## When to use

Use when running trajectory inference on a ~1.3-million-cell single-cell atlas (e.g., MOCA) and needing to confirm scalable runtime (~40 min) plus globally connected, multi-branch topology robustness across PC counts.

## When NOT to use

- Input is a small (~10^2-10^3 cell) simulated or toy dataset — dyntoy/dyngen simulations are for prototyping TI methods, not for this million-cell scalability benchmark
- The goal is topology discovery on data where UMAP/t-SNE pre-embedding suffices — the article notes methods requiring UMAP (or t-SNE) before parsing MOCA are highly susceptible to user-defined input parameters
- Data are disconnected trajectories needing multiple roots — use via_wrapper_disconnected instead of the standard atlas-scale connected workflow

## Inputs

- 1.3-million-cell MOCA mouse organogenesis atlas expression matrix (GEO deposit)
- Principal component embedding stored as adata.obsm['X_pca']
- Cell-type annotations / true labels for cluster coloring and root selection
- UMAP or PHATE embedding for single-cell visualization

## Outputs

- VIA cluster-graph with terminal states, pseudotime and multi-branch adjacency over nine major trajectories
- Per-method runtime table (VIA ~40 min vs PAGA 3 h, Palantir >4 h, STREAM >6.5 h)
- Per-PC topology-persistence check results (connected-component count, branch structure, terminal-state correspondence)

## How to apply

Load the MOCA expression matrix from its GEO deposit into an AnnData object with principal components stored in adata.obsm['X_pca'], then run pyVIA (github.com/ShobiStassen/VIA) using PARC-based clustering with the documented defaults for edgepruning_clustering_resolution (~0.15, i.e., jac_std_global), too_big_factor 0.4, and pseudotime_threshold_TS 30, timing end-to-end execution. Set a root cell via root_user (e.g., root=[42]) and run through via.via_wrapper with the UMAP or PHATE embedding used only for single-cell-level visualization. Run Palantir, STREAM and Monocle3 on the same atlas to record comparator runtimes (reported as 3-4 h for the next fastest method). Then rerun VIA across several values of ncomps, exporting the VIA cluster-graph after each run and checking that the topology remains a single connected component with concordant terminal states and branch adjacency. Assemble per-method runtimes and per-PC topology checks (connected-component count, branch structure, terminal-state correspondence) into one comparison table.

## Related tools

- **VIA** (Primary scalable trajectory inference method; lazy-teleporting random walks over a PARC-clustered graph to compute pseudotime and lineages on the MOCA atlas) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm used inside VIA to build the single-cell cluster-graph nodes prior to trajectory computation)
- **Palantir** (Baseline TI method timed against VIA's ~40 min runtime on MOCA (>4 h))
- **STREAM** (Baseline TI method timed against VIA on MOCA (>6.5 h))
- **Monocle3** (UMAP-dependent comparator whose fine-grained supergroups lose connectivity on MOCA across PC choices)
- **UMAP** (Embedding used for single-cell-level visualization of the VIA trajectory, not for the TI computation itself)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='mouse_organogenesis', too_big_factor=0.4, pseudotime_threshold_TS=30)
```

## Evaluation signals

- VIA end-to-end runtime is on the order of tens of minutes (~40 min scale) while the next fastest comparator exceeds it by hours (3-4 h reported)
- VIA resolves nine major trajectories on the MOCA atlas with fine-grained supergroups kept connected
- Across varying ncomps, the exported cluster-graph remains a single connected component (connected-component count = 1)
- Terminal states and neighborhood/adjacency relationships between cluster nodes are concordant across all tested PC choices
- Monocle3 on the same PCs breaks fine-grained supergroups, providing a negative control for the connectivity check

## Limitations

- The article does not report specific parameter ranges or settings used in the MOCA 'stress tests', so exact reproduction requires reading the paper's supplementary material
- The MOCA dataset is referenced without an accession or repository identifier in the discussion; the GEO deposit must be located independently
- The 'next fastest method' used as runtime comparator (PAGA at 3 h) is not named in the discussion and must be inferred from the results section
- Methods requiring UMAP (or t-SNE) before parsing MOCA are highly susceptible to user-defined input parameters, so comparator behavior is parameter-sensitive

## Evidence

- [discussion] We also demonstrated on the 1.3 million MOCA dataset that VIA is highly scalable with a runtime of ~40 min: "We also demonstrated on the 1.3 million MOCA dataset that VIA is highly scalable with a runtime of ~40 min"
- [intro] Showcase scalability on MOCA: "We showcase the scalability of sample size by analyzing the fine-grained developmental sub-trajectories in the 1.3-million-cell mouse organogenesis atlas"
- [discussion] Methods needing UMAP/t-SNE fail on MOCA: "We showed that methods which require UMAP (or t-SNE) before parsing MOCA are highly susceptible to user defined input parameters"
- [intro] PARC clustering builds the graph: "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
- [methods] AnnData with PCA input format: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] too_big_factor default: "(optional, default = 0.4) if a cluster exceeds this share of the entire cell population, then PARC will be run on the large cluster to increase granularity."
- [abstract] Global connectivity preserved on atlas: "VIA robustly and efficiently unravels the fine-grained sub-trajectories in a 1.3-million-cell transcriptomic mouse atlas without losing the global connectivity"
