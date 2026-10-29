---
name: disconnected-and-cyclic-trajectory-inference
description: Use when single-cell data is expected to contain disconnected populations,
  cyclic/circular processes, or converging multifurcations that tree-only trajectory
  inference methods would merge or break.
when_to_use_negative:
- Data is expected to form a simple linear or tree-like topology where tree-based
  methods (Slingshot, STREAM) suffice and are well-benchmarked
- Input is already a milestone-network / gold-standard topology rather than single-cell
  expression data
- The goal is realistic simulation of synthetic trajectories — use dyngen instead
  of the simplistic dyntoy model
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Primary trajectory inference engine; lazy-teleporting random walks reconstruct
    disconnected and cyclic topologies via via_wrapper / via_wrapper_disconnected
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm that builds the cell-level cluster-graph underlying VIA's
    trajectory computation
  repo: https://github.com/ShobiStassen/VIA
- name: dynverse/dyntoy
  role: Generates the gold-standard synthetic trajectory datasets (Multifurc1, Cyclic1,
    Disconn1, etc.) used to benchmark topology recovery
  repo: https://github.com/dynverse/dyntoy
- name: Palantir
  role: Baseline tree/fate-detection method compared against VIA on the dyntoy datasets
- name: SlingShot
  role: Baseline tree-based TI method that lacks generalizability to disconnected/cyclic
    topologies
- name: STREAM
  role: Baseline tree-based TI method compared in the dynverse metric benchmark
- name: PAGA
  role: Baseline graph-abstraction method that merges elusive terminal states on multifurcation
    data
- name: Monocle3
  role: Baseline TI method compared on the nine dyntoy synthetic datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:1c7513571a92507c5a5de62000d842a8c799a2cf7610e8e810f60e48c4295e9f
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# disconnected-and-cyclic-trajectory-inference

## Summary

Infers non-tree single-cell trajectory topologies — disconnected components, cyclic paths, and hybrid structures — using VIA's lazy-teleporting random walks, which tree-based methods like Slingshot and STREAM cannot represent.

## When to use

Use when single-cell data is expected to contain disconnected populations, cyclic/circular processes, or converging multifurcations that tree-only trajectory inference methods would merge or break.

## When NOT to use

- Data is expected to form a simple linear or tree-like topology where tree-based methods (Slingshot, STREAM) suffice and are well-benchmarked
- Input is already a milestone-network / gold-standard topology rather than single-cell expression data
- The goal is realistic simulation of synthetic trajectories — use dyngen instead of the simplistic dyntoy model

## Inputs

- Single-cell expression matrix as AnnData with PCA stored in adata.obsm['X_pca']
- dyntoy synthetic trajectory datasets (e.g., Multifurc1, Cyclic1, Disconn1, ~1000 cells) with gold-standard topology, pseudotime, and cell-fate labels
- Cell-level embedding (UMAP or PHATE) for visualization
- Optional root/start cell indices or root cluster labels (e.g., root=[42] connected; root=[1,1] disconnected)

## Outputs

- Inferred trajectory graph with terminal states and branch assignments per cell
- Pseudotime values per cell
- Cell-fate/lineage probabilities including elusive terminal states (e.g., the M2 lineage on the four-leaf multifurcation)
- dyntoy TI benchmark metrics table (e.g., dyntoy_ti_benchmark_metrics.csv) with GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, and F1 cell fate

## How to apply

Reduce the data to its first ~10 principal components (AnnData with PCA in adata.obsm['X_pca']) and compute a UMAP or PHATE embedding for visualization. For connected cyclic or multifurcating topologies, run VIA via via.via_wrapper with a knn around 20 and a fixed random seed, setting the root via root_user (e.g., root=[42]). For disconnected topologies, use via.via_wrapper_disconnected with multiple roots such as root=[1,1] and preserve_disconnected=True so separate components are not falsely joined. The rationale is VIA's cluster-graph (built with PARC clustering) plus lazy-teleporting random walks with MCMC refinement, which generalizes beyond tree-like pathways where Slingshot, STREAM, Monocle3, PAGA, and Palantir exhibit low sensitivity to cell fates. Validate against a gold standard (e.g., dyntoy milestone networks) using the five dynverse metrics: graph edit distance, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate.

## Related tools

- **VIA** (Primary trajectory inference engine; lazy-teleporting random walks reconstruct disconnected and cyclic topologies via via_wrapper / via_wrapper_disconnected) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm that builds the cell-level cluster-graph underlying VIA's trajectory computation) — https://github.com/ShobiStassen/VIA
- **dynverse/dyntoy** (Generates the gold-standard synthetic trajectory datasets (Multifurc1, Cyclic1, Disconn1, etc.) used to benchmark topology recovery) — https://github.com/dynverse/dyntoy
- **Palantir** (Baseline tree/fate-detection method compared against VIA on the dyntoy datasets)
- **SlingShot** (Baseline tree-based TI method that lacks generalizability to disconnected/cyclic topologies)
- **STREAM** (Baseline tree-based TI method compared in the dynverse metric benchmark)
- **PAGA** (Baseline graph-abstraction method that merges elusive terminal states on multifurcation data)
- **Monocle3** (Baseline TI method compared on the nine dyntoy synthetic datasets)

## Examples

```
import via; v = via.via_wrapper_disconnected(adata, true_label, embedding=adata.obsm['X_pca'][:, 0:10], knn=20, root=[1,1], preserve_disconnected=True)
```

## Evaluation signals

- On the four-leaf multifurcation dyntoy dataset, VIA must detect the elusive M2 terminal state that Palantir, PAGA, Slingshot, STREAM, and Monocle3 all merge with the M8 lineage
- The cyclic (Cyclic1) and disconnected (Disconn1) topologies must be recovered — disconnected components remain separate with preserve_disconnected=True rather than being falsely joined
- Every inferred trajectory scores against the dyntoy gold standard on all five dynverse metrics: GED, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate
- VIA should attain the best composite accuracy across the five metrics relative to all baselines before results are saved (e.g., dyntoy_ti_benchmark_metrics.csv)
- Inferred pseudotime and branch labels should be stable across runs given a fixed random seed and consistent knn (e.g., knn=20)

## Limitations

- Tree-based baselines (Slingshot, STREAM, Monocle3, PAGA, Palantir) exhibit low sensitivity to cell fates and merge elusive terminal lineages, so comparisons should not treat them as topology-generalizable references
- dyntoy's generative model is very simplistic and far from realistic; for more realistic synthetic data the dynverse authors recommend dyngen
- VIA's tuning parameters (e.g., edgepruning_clustering_resolution, jac_std_global, pseudotime_threshold_TS, too_big_factor) affect connectivity and granularity and specific stress-test parameter ranges are not fully reported in the discussion

## Evidence

- [abstract] using lazy-teleporting random walks to accurately reconstruct complex cellular trajectories beyond tree-like pathways (e.g., cyclic or disconnected structures): "using lazy-teleporting random walks to accurately reconstruct complex cellular trajectories beyond tree-like pathways (e.g., cyclic or disconnected structures)"
- [intro] current TI methods predominantly work well on tree-like trajectories but lack generalizability: "current trajectory inference (TI) methods predominantly work well on tree-like trajectories (e.g., Slingshot and STREAM), but lack the generalizability to infer disconnected, cyclic or hybrid"
- [methods] via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [methods] call VIA and set an early start cell via root_user: "# call VIA. We identify an early (suitable) start cell root = [42]. Can also set an arbitrary value"
- [readme] dyntoy simulates single-cell expression data in which a single-cell trajectory is present, including loops: "dyntoy simulates single-cell expression data in which a single-cell trajectory is present. Even though the model to generate the data is
very simplistic (and far from realistic), it can simulate very"
- [readme] Multifurcation starter tutorial uses the 4-leaf DynToy simulation: [evidence span withheld — anchor status: not_found]
