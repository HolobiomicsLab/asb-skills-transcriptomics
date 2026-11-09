---
name: runtime-benchmarking-of-ti-methods
description: Decision support for choosing a TI method when the input atlas exceeds
  ~10^5–10^6 cells (e.g. MOCA-style organogenesis data) and runtime plus preservation
  of global connectivity across parameter choices must be compared against baselines
  like PAGA, Palantir, STREAM and Monocle3.
when_to_use_negative:
- The dataset is small (~10^2–10^3 cells) or simulated toy data, where runtime scalability
  is not the bottleneck and methods like Slingshot or STREAM suffice
- The goal is topology accuracy on disconnected, cyclic or hybrid trajectories rather
  than runtime/memory benchmarking at atlas scale
- Input requires UMAP or t-SNE embedding first — the article notes UMAP/t-SNE-based
  methods like Monocle3 are highly susceptible to user-defined input parameters on
  MOCA
edam_operation: http://edamontology.org/operation_3890
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Primary scalable TI method benchmarked; lazy-teleporting random walks on a
    PARC cluster-graph, ~40 min on 1.3M cells
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm that builds the single-cell graph so each VIA cluster-graph
    node represents a cluster of cells
  repo: https://github.com/ShobiStassen/VIA
- name: Palantir
  role: Baseline TI method whose runtime (>4 h on MOCA) is recorded for comparison
- name: STREAM
  role: Baseline TI method whose runtime (>6.5 h on MOCA) is recorded for comparison
- name: Monocle3
  role: UMAP-based baseline that loses fine-grained supergroup connectivity on MOCA
    across PC choices
- name: UMAP
  role: Embedding used for single-cell visualization of VIA trajectories; identified
    as a failure point for Monocle3 on MOCA
- name: dyntoy
  role: Simulated toy trajectory generator (e.g. 4-leaf multifurcation) for prototyping
    and testing TI pipelines before atlas-scale runs
  repo: dynverse/dyntoy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:da8261eb96ffef7636fd522aa021037eeaacadbe454329927c990bc181068e05
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
  - dyntoy
  edam_operation: http://edamontology.org/operation_3890
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# runtime-benchmarking-of-ti-methods

## Summary

Benchmark the end-to-end runtime and topology robustness of single-cell trajectory inference (TI) methods at atlas scale, as done when VIA resolved nine major trajectories in the 1.3-million-cell MOCA mouse organogenesis atlas in ~40 min versus 3–6.5 h for PAGA, Palantir and STREAM.

## When to use

Decision support for choosing a TI method when the input atlas exceeds ~10^5–10^6 cells (e.g. MOCA-style organogenesis data) and runtime plus preservation of global connectivity across parameter choices must be compared against baselines like PAGA, Palantir, STREAM and Monocle3.

## When NOT to use

- The dataset is small (~10^2–10^3 cells) or simulated toy data, where runtime scalability is not the bottleneck and methods like Slingshot or STREAM suffice
- The goal is topology accuracy on disconnected, cyclic or hybrid trajectories rather than runtime/memory benchmarking at atlas scale
- Input requires UMAP or t-SNE embedding first — the article notes UMAP/t-SNE-based methods like Monocle3 are highly susceptible to user-defined input parameters on MOCA

## Inputs

- 1.3-million-cell MOCA mouse organogenesis atlas expression matrix (GEO deposit)
- AnnData object with principal components in adata.obsm['X_pca']
- Number-of-components setting (ncomps) to sweep, e.g. default ncomps=20

## Outputs

- Per-method end-to-end runtime table (VIA ~40 min vs PAGA 3 h, Palantir >4 h, STREAM >6.5 h)
- VIA cluster-graph per PC setting (terminal states, adjacency between cluster nodes)
- Topology-persistence report: connected-component count, branch structure, terminal-state correspondence across PC choices

## How to apply

Load the atlas (e.g. the 1.3M-cell MOCA expression matrix from its GEO deposit) as AnnData with principal components stored in adata.obsm['X_pca'], and run VIA (pyVIA) with PARC-based clustering using documented defaults: edgepruning_clustering_resolution ~0.15, too_big_factor 0.4, pseudotime_threshold_TS 30, timing end-to-end execution. Run the baseline TI methods (PAGA, Palantir, STREAM, Monocle3) on the same input PCs and record their runtimes for direct comparison. Then rerun VIA across several values of ncomps (number of principal components) and after each run export the VIA cluster-graph to verify the globally connected, multi-branch topology is preserved. Assemble per-method runtimes and per-PC topology-persistence checks (connected-component count, branch structure, terminal-state correspondence) into a single comparison table.

## Related tools

- **VIA** (Primary scalable TI method benchmarked; lazy-teleporting random walks on a PARC cluster-graph, ~40 min on 1.3M cells) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm that builds the single-cell graph so each VIA cluster-graph node represents a cluster of cells) — https://github.com/ShobiStassen/VIA
- **Palantir** (Baseline TI method whose runtime (>4 h on MOCA) is recorded for comparison)
- **STREAM** (Baseline TI method whose runtime (>6.5 h on MOCA) is recorded for comparison)
- **Monocle3** (UMAP-based baseline that loses fine-grained supergroup connectivity on MOCA across PC choices)
- **UMAP** (Embedding used for single-cell visualization of VIA trajectories; identified as a failure point for Monocle3 on MOCA)
- **dyntoy** (Simulated toy trajectory generator (e.g. 4-leaf multifurcation) for prototyping and testing TI pipelines before atlas-scale runs) — dynverse/dyntoy

## Examples

```
conda create --name ViaEnv python=3.10; pip install pyVIA; via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42])
```

## Evaluation signals

- VIA runtime on the 1.3M-cell MOCA atlas is on the order of tens of minutes (~40 min) while every baseline exceeds it by hours (next fastest 3–4 h)
- After each ncomps sweep run, the exported VIA cluster-graph remains a single connected component with concordant three-branch adjacency structure
- Fine-grained supergroups stay connected and neighborhood relationships are preserved across the tested range of principal components, unlike UMAP-based Monocle3
- Nine major developmental trajectories are recovered, matching the reported MOCA result
- Per-method runtimes and per-PC topology checks assemble into a single reproducible comparison table

## Limitations

- The 'next fastest method' used as the runtime comparator is not explicitly named in the discussion (3–4 h)
- The MOCA dataset is referenced without an accession or repository identifier
- No software version numbers are stated for VIA or the compared methods
- Specific parameter ranges used in the topology 'stress tests' are not reported in the discussion; results defer to supplementary figures
- Methods requiring UMAP or t-SNE before parsing MOCA are highly susceptible to user-defined input parameters

## Evidence

- [discussion] Scalability runtime finding: "We also demonstrated on the 1.3 million MOCA dataset that VIA is highly scalable with a runtime of ~40 min"
- [discussion] Comparator runtime: "VIA is highly scalable with a runtime of ~40 min (compared to 3–4 h on the next fastest method)"
- [abstract] Global connectivity preservation: "VIA robustly and efficiently unravels the fine-grained sub-trajectories in a 1.3-million-cell transcriptomic mouse atlas without losing the global connectivity"
- [methods] Data input format: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] Key tuning parameter: "global level  graph pruning for PARC clustering stage. Key tuning parameter."
- [discussion] UMAP susceptibility: "We showed that methods which require UMAP (or t-SNE) before parsing MOCA are highly susceptible to user defined input parameters"
- [intro] Scalability claim intro: "VIA is highly scalable with respect to number of cells (102 to >106 cells) and features, without requiring extensive dimensionality reduction or subsampling which compromise global information"
