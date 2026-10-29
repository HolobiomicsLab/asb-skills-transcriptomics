---
name: large-scale-scrna-trajectory-inference
description: Use when analyzing a million-scale single-cell transcriptomic atlas (e.g.,
  1.3M cells stored as PCs in adata.obsm['X_pca']) and you need trajectory inference,
  automated terminal-state detection, and runtime comparison against PAGA/Palantir/STREAM/Monocle3
  without losing global connectivity.
when_to_use_negative:
- The input is small (<100k cells) with a simple tree-like topology, where existing
  methods like Slingshot or STREAM already work well per the paper's framing
- The data is not a single-cell expression/feature matrix reducible to PCs (e.g.,
  already aggregated cluster-level counts or a feature table), since VIA builds a
  cell-cell graph via PARC clustering
- The analysis requires a UMAP- or t-SNE-embedding-driven trajectory (e.g., Monocle3-style),
  which the paper shows is highly susceptible to user-defined input parameters on
  MOCA
tools:
- name: VIA
  role: 'Primary trajectory inference algorithm: builds a cluster-graph of cells via
    PARC, computes pseudotime and reconstructs lineages via lazy-teleporting random
    walks with MCMC refinement'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm that builds the single-cell graph where each node represents
    a cluster of single cells, forming the basis of VIA's cluster-graph
- name: Palantir
  role: Baseline TI method whose runtime (>4 h on MOCA) is compared against VIA's
    ~40 min
- name: STREAM
  role: Baseline TI method whose runtime (>6.5 h on MOCA) is compared against VIA
- name: Monocle3
  role: UMAP-based baseline whose topology on MOCA loses fine-grained supergroup connectivity
    across choices of input PCs, unlike VIA
- name: PAGA
  role: Baseline partition-based graph abstraction method with ~3 h runtime on the
    1.3M-cell atlas used in the runtime comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:d35a63991630262de099ceb9edb98174902ba7ed48bad1eeae0be2627f13141d
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
  - VIA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# large-scale-scrna-trajectory-inference

## Summary

Benchmarks VIA (pyVIA), a lazy-teleporting random-walk trajectory inference method, on the 1.3-million-cell MOCA mouse organogenesis atlas to show scalable pseudotime and lineage reconstruction (~40 min) with preserved global connectivity, against PAGA, Palantir, STREAM and Monocle3 baselines.

## When to use

Use when analyzing a million-scale single-cell transcriptomic atlas (e.g., 1.3M cells stored as PCs in adata.obsm['X_pca']) and you need trajectory inference, automated terminal-state detection, and runtime comparison against PAGA/Palantir/STREAM/Monocle3 without losing global connectivity.

## When NOT to use

- The input is small (<100k cells) with a simple tree-like topology, where existing methods like Slingshot or STREAM already work well per the paper's framing
- The data is not a single-cell expression/feature matrix reducible to PCs (e.g., already aggregated cluster-level counts or a feature table), since VIA builds a cell-cell graph via PARC clustering
- The analysis requires a UMAP- or t-SNE-embedding-driven trajectory (e.g., Monocle3-style), which the paper shows is highly susceptible to user-defined input parameters on MOCA

## Inputs

- 1.3-million-cell MOCA mouse organogenesis scRNA-seq expression matrix from GEO deposit
- AnnData object with principal components stored in adata.obsm['X_pca']
- Root/start cell annotation (root_user) or an arbitrary early cell index
- Same atlas in a format consumable by baseline TI methods (Palantir, STREAM, Monocle3, PAGA)

## Outputs

- VIA cluster-graph with detected terminal states and multi-branch adjacency relationships
- Pseudotime values and reconstructed cell lineages for ~1.3M cells
- Per-method runtime table (VIA ~40 min vs PAGA 3 h, Palantir >4 h, STREAM >6.5 h)
- Per-PC-count topology-persistence check results (connected-component count, branch structure, terminal-state correspondence)

## How to apply

Load the 1.3-million-cell MOCA expression matrix from its GEO deposit and store principal components as the VIA input via adata.obsm['X_pca']. Run VIA (pyVIA) with PARC-based clustering of the single-cell graph using documented defaults: edgepruning_clustering_resolution ~0.15, too_big_factor 0.4, pseudotime_threshold_TS 30, small_pop 10; set an early root cell via root_user and time end-to-end execution. Run Palantir, STREAM and Monocle3 (and PAGA) on the same atlas and record runtimes for direct comparison (~40 min for VIA vs 3-4 h for the next fastest, 3 h PAGA, >4 h Palantir, >6.5 h STREAM). Stress-test robustness by rerunning VIA across several numbers of principal components (varying ncomps) and exporting the VIA cluster-graph after each run to verify terminal states and cluster-node adjacency are preserved. Assemble per-method runtimes and per-PC topology checks (connected-component count, branch structure, terminal-state correspondence) into a single comparison table.

## Related tools

- **VIA** (Primary trajectory inference algorithm: builds a cluster-graph of cells via PARC, computes pseudotime and reconstructs lineages via lazy-teleporting random walks with MCMC refinement) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm that builds the single-cell graph where each node represents a cluster of single cells, forming the basis of VIA's cluster-graph)
- **Palantir** (Baseline TI method whose runtime (>4 h on MOCA) is compared against VIA's ~40 min)
- **STREAM** (Baseline TI method whose runtime (>6.5 h on MOCA) is compared against VIA)
- **Monocle3** (UMAP-based baseline whose topology on MOCA loses fine-grained supergroup connectivity across choices of input PCs, unlike VIA)
- **PAGA** (Baseline partition-based graph abstraction method with ~3 h runtime on the 1.3M-cell atlas used in the runtime comparison)

## Examples

```
via.via_wrapper(adata, true_label, embedding=adata.obsm['X_umap'], knn=10, ncomps=20, jac_std_global=0.15, root=[42])
```

## Evaluation signals

- VIA's end-to-end runtime on the 1.3M-cell MOCA atlas is on the order of tens of minutes (~40 min), while every baseline exceeds it by hours (3-4 h for the next fastest)
- After each rerun across varying ncomps, the VIA cluster-graph remains a single globally connected component
- Fine-grained supergroups stay connected and neighborhood relationships are preserved across all tested PC choices, unlike UMAP-based Monocle3
- Terminal states and cluster-node adjacency relationships are concordant across PC-count reruns (e.g., the nine major MOCA trajectories)
- VIA parameters match documented defaults: edgepruning_clustering_resolution ~0.15, too_big_factor 0.4, pseudotime_threshold_TS 30

## Limitations

- The discussion does not name the 'next fastest method' used as the 3-4 h runtime comparator, and states no software version numbers for VIA or compared methods
- The MOCA dataset is referenced without an accession or repository identifier, complicating direct data retrieval
- Methods requiring UMAP (or t-SNE) before parsing MOCA are highly susceptible to user-defined input parameters, so embedding-choice effects must be controlled when comparing
- Specific parameter ranges and settings used in the paper's 'stress tests' are not reported in the discussion

## Evidence

- [abstract] VIA robustly and efficiently unravels fine-grained sub-trajectories in a 1.3-million-cell transcriptomic mouse atlas without losing global connectivity: "VIA robustly and efficiently unravels the fine-grained sub-trajectories in a 1.3-million-cell transcriptomic mouse atlas without losing the global connectivity"
- [discussion] MOCA scalability runtime: "We also demonstrated on the 1.3 million MOCA dataset that VIA is highly scalable with a runtime of ~40 min (compared to 3–4 h on the next fastest method)"
- [methods] AnnData PCA input format: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [intro] Baseline method sensitivity: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] VIA wrapper invocation with key parameters: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [intro] PARC clustering of single-cell graph: "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
