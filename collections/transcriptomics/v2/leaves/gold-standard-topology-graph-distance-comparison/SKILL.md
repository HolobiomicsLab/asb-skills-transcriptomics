---
name: gold-standard-topology-graph-distance-comparison
description: Use when comparing a trajectory inference method's inferred topology,
  pseudotime, and cell-fate predictions against known synthetic ground truth (e.g.,
  dyntoy multifurcating, cyclic, or disconnected datasets) to quantify accuracy.
when_to_use_negative:
- Input is real biological data without known gold-standard topology/pseudotime —
  the dynverse metrics require ground-truth milestone networks and fate labels
- The goal is realistic synthetic data rather than quick topology benchmarking — the
  dyntoy README notes its model is 'very simplistic (and far from realistic)' and
  recommends dyngen for more realistic simulations
- Datasets are atlas-scale (10^5–10^6 cells) where runtime/scalability, not gold-standard
  topology accuracy, is the evaluation target
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Candidate trajectory inference method scored against the dyntoy gold standard;
    run via via.via_wrapper (connected) and via.via_wrapper_disconnected (Disconn1)
  repo: https://github.com/ShobiStassen/VIA
- name: dynverse/dyntoy
  role: Generator of the nine synthetic trajectory datasets with gold-standard topology,
    pseudotime, and cell-fate labels used as benchmark ground truth
  repo: https://github.com/dynverse/dyntoy
- name: Palantir
  role: Baseline TI method scored with the same five dynverse metrics for composite
    accuracy comparison
- name: PAGA
  role: Baseline TI method scored with the same five dynverse metrics for composite
    accuracy comparison
- name: Slingshot
  role: Baseline TI method scored with the same five dynverse metrics for composite
    accuracy comparison
- name: STREAM
  role: Baseline TI method scored with the same five dynverse metrics for composite
    accuracy comparison
- name: Monocle3
  role: Baseline TI method scored with the same five dynverse metrics for composite
    accuracy comparison
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph construction used in the
    benchmarked pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:0a6c79a229e4dba9ee348446cb3827a122bd703180a0635ed44c9bf8ae1a262c
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
  - Slingshot
  - VIA
  - dynverse/dyntoy
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gold-standard-topology-graph-distance-comparison

## Summary

Benchmark a trajectory inference method against gold-standard synthetic topologies (dynverse/dyntoy) by scoring inferred trajectories with the five dynverse metrics — graph edit distance, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate — and aggregating them into a composite accuracy ranking, as done for VIA versus Palantir, PAGA, Slingshot, STREAM, and Monocle3.

## When to use

Use when comparing a trajectory inference method's inferred topology, pseudotime, and cell-fate predictions against known synthetic ground truth (e.g., dyntoy multifurcating, cyclic, or disconnected datasets) to quantify accuracy.

## When NOT to use

- Input is real biological data without known gold-standard topology/pseudotime — the dynverse metrics require ground-truth milestone networks and fate labels
- The goal is realistic synthetic data rather than quick topology benchmarking — the dyntoy README notes its model is 'very simplistic (and far from realistic)' and recommends dyngen for more realistic simulations
- Datasets are atlas-scale (10^5–10^6 cells) where runtime/scalability, not gold-standard topology accuracy, is the evaluation target

## Inputs

- Nine dyntoy synthetic single-cell trajectory datasets (~1000 cells, 1000 features each: Multifurc1, Cyclic1, Disconn1, Conn1, etc.) with gold-standard topology, pseudotime, and cell-fate labels
- Inferred trajectories (topology, pseudotime, terminal/fate assignments) from VIA and baselines Palantir, PAGA, Slingshot, STREAM, Monocle3

## Outputs

- Per-dataset scores for the five dynverse metrics: graph edit distance (GED), F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, F1 cell fate
- Composite accuracy table ranking VIA against all baselines (e.g., dyntoy_ti_benchmark_metrics.csv)

## How to apply

Generate the dyntoy synthetic trajectory datasets (e.g., Multifurc1/four-leaf, Cyclic1, Disconn1, Conn1; 1000 cells each) with dynverse/dyntoy via generate_dataset, retaining the gold-standard milestone_network topology, pseudotime, and cell-fate labels. Reduce each dataset to its first 10 PCs and run the candidate method — for VIA use via.via_wrapper (knn=20, fixed random_seed) for connected topologies and via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True for Disconn1. Run baselines (Monocle3, Slingshot, STREAM, PAGA, Palantir) on the same nine datasets with default or benchmark-prescribed settings. Score every inferred trajectory against the dyntoy gold standard using the five dynverse metrics: graph edit distance (GED), F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate. Aggregate per-dataset metric means into a composite accuracy table ranking the candidate method against all baselines, e.g. saved as dyntoy_ti_benchmark_metrics.csv.

## Related tools

- **VIA** (Candidate trajectory inference method scored against the dyntoy gold standard; run via via.via_wrapper (connected) and via.via_wrapper_disconnected (Disconn1)) — https://github.com/ShobiStassen/VIA
- **dynverse/dyntoy** (Generator of the nine synthetic trajectory datasets with gold-standard topology, pseudotime, and cell-fate labels used as benchmark ground truth) — https://github.com/dynverse/dyntoy
- **Palantir** (Baseline TI method scored with the same five dynverse metrics for composite accuracy comparison)
- **PAGA** (Baseline TI method scored with the same five dynverse metrics for composite accuracy comparison)
- **Slingshot** (Baseline TI method scored with the same five dynverse metrics for composite accuracy comparison)
- **STREAM** (Baseline TI method scored with the same five dynverse metrics for composite accuracy comparison)
- **Monocle3** (Baseline TI method scored with the same five dynverse metrics for composite accuracy comparison)
- **PARC** (Clustering algorithm underlying VIA's cluster-graph construction used in the benchmarked pipeline)

## Examples

```
via.via_wrapper_disconnected(adata, true_label, embedding=adata.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True)
```

## Evaluation signals

- VIA detects the elusive M2 terminal state on the four-leaf multifurcation dataset, while Palantir, PAGA, Slingshot, STREAM, and Monocle3 merge it with the M8 lineage
- VIA correctly infers the cyclic (Cyclic1) and disconnected (Disconn1) topologies that tree-based baselines miss
- VIA attains the best composite accuracy when the five dynverse metric means (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate) are aggregated across all nine datasets
- Each dataset contains the expected 1000 cells and the gold-standard milestone_network (e.g., directed edges M1->M3, M3->M4, M3->M2) is recovered in the comparison
- Metrics table is saved as dyntoy_ti_benchmark_metrics.csv with per-dataset metric means for every method

## Limitations

- dyntoy's generative model is very simplistic and far from realistic; it is suited for quick prototyping but more realistic synthetic data requires the dyngen package
- The article's discussion reports no quantitative accuracy metrics inline — results defer to supplementary figures — and no software version numbers are stated for VIA or the compared methods
- Comparison methods requiring UMAP or t-SNE embeddings first are highly susceptible to user-defined input parameters, which can confound benchmark fairness

## Evidence

- [results] VIA detects the elusive M2 terminal state on the four-leaf dataset where baselines merge it with M8: "using lazy-teleporting random walks to accurately reconstruct complex cellular trajectories beyond tree-like pathways (e.g., cyclic or disconnected structures)"
- [readme] dyntoy data is simplistic and intended for prototyping TI methods: [evidence span withheld — anchor status: external_doc]
- [methods] VIA run for disconnected topologies uses via_wrapper_disconnected with preserve_disconnected=True: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [intro] Existing fate-detecting methods show low sensitivity to cell fates: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [readme] dyntoy generates datasets with milestone network ground truth via generate_dataset: "dataset <- generate_dataset(
  model = model_bifurcating(),
  num_cells = 1000,
  num_features = 1000
)"
