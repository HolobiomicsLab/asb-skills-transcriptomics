---
name: cell-fate-terminal-state-detection-evaluation
description: Use when a single-cell trajectory dataset may harbor elusive or low-population
  terminal fates (e.g., multifurcating, cyclic, or disconnected topologies) and you
  need automated terminal-state calling plus a quantitative fate-recovery score against
  known labels.
when_to_use_negative:
- When the goal is only a static cell-type clustering with no trajectory, root, or
  fate structure — PARC alone or standard clustering suffices
- When no root/early cell can be proposed and no gold-standard or time-series labels
  exist, since terminal-state calling depends on pseudotime orientation from a root
- When the input is a feature/abundance table or bulk data rather than single-cell-level
  observations, which the cluster-graph random-walk framework requires
tools:
- name: VIA
  role: 'Primary algorithm: lazy-teleporting random walks on a PARC cluster-graph
    to compute pseudotime, call terminal states, and reconstruct cell lineages'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering stage that builds the single-cell cluster-graph (with edgepruning_clustering_resolution
    and jac_std_global tuning) underlying VIA's trajectory computations
- name: dynverse/dyntoy
  role: Generates synthetic benchmark datasets (e.g., 4-leaf multifurcation, 1000
    cells) with gold-standard topology and fate labels for scoring terminal-fate recovery
  repo: https://github.com/dynverse/dyntoy
- name: Palantir
  role: Baseline fate-detection method compared against VIA for terminal-state sensitivity
- name: PAGA
  role: Baseline trajectory/fate method used in the dynverse benchmark comparison
- name: SlingShot
  role: Baseline tree-based trajectory method benchmarked on fate detection
- name: STREAM
  role: Baseline tree-based trajectory method benchmarked on fate detection
- name: Monocle3
  role: Baseline trajectory method benchmarked on fate detection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:c6586d44dfa4523b609b46805d32c60750a0297b4975c616d52ebd7aad4ec34e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PAGA
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - VIA
  - dynverse/dyntoy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-fate-terminal-state-detection-evaluation

## Summary

Detect terminal cell states and rare terminal fates with VIA's lazy-teleporting random-walk lineage inference, and evaluate fate detection against a gold standard using dynverse F1 cell-fate (and branch) metrics. This addresses the documented low sensitivity of Palantir, PAGA, Slingshot, STREAM, and Monocle3 to less populous cell fates.

## When to use

Use when a single-cell trajectory dataset may harbor elusive or low-population terminal fates (e.g., multifurcating, cyclic, or disconnected topologies) and you need automated terminal-state calling plus a quantitative fate-recovery score against known labels.

## When NOT to use

- When the goal is only a static cell-type clustering with no trajectory, root, or fate structure — PARC alone or standard clustering suffices
- When no root/early cell can be proposed and no gold-standard or time-series labels exist, since terminal-state calling depends on pseudotime orientation from a root
- When the input is a feature/abundance table or bulk data rather than single-cell-level observations, which the cluster-graph random-walk framework requires

## Inputs

- Single-cell expression matrix as AnnData with PCA stored in adata.obsm['X_pca'] (e.g., first 10-20 PCs)
- dyntoy synthetic trajectory dataset with gold-standard milestone network, pseudotime, and cell-fate labels (1000 cells, 1000 features)
- Cell group labels (true_label) and a 2D embedding (UMAP/PHate) for visualization

## Outputs

- Predicted terminal state assignments and lineage/fate probabilities per cell from VIA
- Inferred cluster-level trajectory graph with pseudotime
- dyntoy_ti_benchmark_metrics.csv with per-dataset GED, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate scores

## How to apply

Reduce the expression matrix to its first ~10-20 PCs (store in adata.obsm['X_pca']) and run VIA via via.via_wrapper for connected topologies (knn=10-20, jac_std_global=0.15, root set via root_user, e.g. root=[42]) or via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True for disconnected components. Terminal states are called automatically using a percentile threshold (pseudotime_threshold_TS, default 30) on node potential; clusters smaller than small_pop (default 10 cells) are merged, and clusters exceeding too_big_factor (default 0.4) are sub-clustered to raise granularity. For synthetic benchmarks, generate dyntoy datasets (e.g., the 4-leaf multifurcation, 1000 cells) retaining gold-standard topology, pseudotime, and fate labels, then score VIA's predicted fates and branches against the gold standard with the five dynverse metrics (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate), comparing against Palantir, PAGA, Slingshot, STREAM, and Monocle3. Success is demonstrated when VIA recovers rare terminal states (e.g., the M2 state on the four-leaf dataset) that baselines merge into other lineages (e.g., M8), and attains the best composite accuracy across the metrics.

## Related tools

- **VIA** (Primary algorithm: lazy-teleporting random walks on a PARC cluster-graph to compute pseudotime, call terminal states, and reconstruct cell lineages) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering stage that builds the single-cell cluster-graph (with edgepruning_clustering_resolution and jac_std_global tuning) underlying VIA's trajectory computations)
- **dynverse/dyntoy** (Generates synthetic benchmark datasets (e.g., 4-leaf multifurcation, 1000 cells) with gold-standard topology and fate labels for scoring terminal-fate recovery) — https://github.com/dynverse/dyntoy
- **Palantir** (Baseline fate-detection method compared against VIA for terminal-state sensitivity)
- **PAGA** (Baseline trajectory/fate method used in the dynverse benchmark comparison)
- **SlingShot** (Baseline tree-based trajectory method benchmarked on fate detection)
- **STREAM** (Baseline tree-based trajectory method benchmarked on fate detection)
- **Monocle3** (Baseline trajectory method benchmarked on fate detection)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='toy_multifurcation')
```

## Evaluation signals

- VIA recovers the elusive M2 terminal state on the dyntoy four-leaf multifurcation dataset, whereas Palantir, PAGA, Slingshot, STREAM, and Monocle3 merge M2 with the M8 lineage
- Terminal-state calls respect the pseudotime_threshold_TS percentile (default 30) on node potential and small_pop (default 10) merging behavior
- F1 cell fate and F1-branch scores computed against the dyntoy gold standard exceed those of all five baselines across the nine synthetic datasets
- Cyclic and disconnected topologies are inferred correctly (via via_wrapper_disconnected with preserve_disconnected=True) where tree-based baselines fail
- Output CSV (dyntoy_ti_benchmark_metrics.csv) contains per-dataset means for all five dynverse metrics: GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, and F1 cell fate

## Limitations

- Methods that require UMAP or t-SNE before parsing large atlases are highly susceptible to user-defined input parameters; embeddings must be chosen carefully
- Small fates are inherently hard: VIA attempts to merge clusters with population < 10 cells (small_pop), so extremely rare terminal fates may be lost if small_pop is too high
- The 'stress tests' and quantitative accuracy metrics are deferred to supplementary figures, with no software versions or parameter ranges reported in the main discussion

## Evidence

- [methods] Automated terminal-state thresholding default: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [methods] Disconnected wrapper usage: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [intro] Baseline low sensitivity to cell fates motivating the skill: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] Root specification for pseudotime orientation: "# call VIA. We identify an early (suitable) start cell root = [42]. Can also set an arbitrary value"
- [readme] Starter tutorial dataset grounding the benchmark use case: [evidence span withheld — anchor status: not_found]
