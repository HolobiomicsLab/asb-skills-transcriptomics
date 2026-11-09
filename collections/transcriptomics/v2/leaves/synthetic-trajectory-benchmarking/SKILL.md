---
name: synthetic-trajectory-benchmarking
description: Use when comparing TI algorithms on simulated single-cell trajectories
  (e.g., multifurcating, cyclic, disconnected, or converging topologies) where dyntoy
  gold-standard milestone/pseudotime/cell-fate labels exist and a composite accuracy
  ranking is needed.
when_to_use_negative:
- When more realistic synthetic single-cell data are required — the dyntoy README
  explicitly directs users to the dyngen package for this
- When the input is real biological data without gold-standard trajectory labels,
  since the dynverse metrics require ground-truth topology, pseudotime, and cell-fate
  annotations
- When evaluating scalability or atlas-scale behavior (e.g., million-cell MOCA) rather
  than topology/cell-fate accuracy on small toy datasets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Candidate TI method benchmarked on the nine dyntoy datasets; run via via.via_wrapper
    (knn=20, fixed seed) for connected topologies and via.via_wrapper_disconnected
    (root=[1,1], preserve_disconnected=True) for disconnected ones
  repo: https://github.com/ShobiStassen/VIA
- name: dynverse/dyntoy
  role: Generates the synthetic single-cell trajectory datasets with gold-standard
    topology, pseudotime, and cell-fate labels used as benchmark ground truth
  repo: https://github.com/dynverse/dyntoy
- name: Palantir
  role: Baseline TI method scored against the dyntoy gold standard with the five dynverse
    metrics
- name: PAGA
  role: Baseline TI method scored against the dyntoy gold standard with the five dynverse
    metrics
- name: Slingshot
  role: Baseline TI method scored against the dyntoy gold standard with the five dynverse
    metrics
- name: STREAM
  role: Baseline TI method scored against the dyntoy gold standard with the five dynverse
    metrics
- name: Monocle3
  role: Baseline TI method scored against the dyntoy gold standard with the five dynverse
    metrics
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph construction during the
    benchmark runs
  repo: https://github.com/ShobiStassen/VIA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:f737e89612a4b70fbd10baa43b584d4898308fe4129fbee9fd64c3eff028b1d5
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# synthetic-trajectory-benchmarking

## Summary

Benchmark single-cell trajectory inference (TI) methods on dynverse/dyntoy synthetic datasets by scoring inferred topologies, pseudotime, and cell fates against gold-standard labels using the five dynverse metrics (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate), as used to evaluate VIA against Palantir, PAGA, Slingshot, STREAM, and Monocle3.

## When to use

Use when comparing TI algorithms on simulated single-cell trajectories (e.g., multifurcating, cyclic, disconnected, or converging topologies) where dyntoy gold-standard milestone/pseudotime/cell-fate labels exist and a composite accuracy ranking is needed.

## When NOT to use

- When more realistic synthetic single-cell data are required — the dyntoy README explicitly directs users to the dyngen package for this
- When the input is real biological data without gold-standard trajectory labels, since the dynverse metrics require ground-truth topology, pseudotime, and cell-fate annotations
- When evaluating scalability or atlas-scale behavior (e.g., million-cell MOCA) rather than topology/cell-fate accuracy on small toy datasets

## Inputs

- dynverse/dyntoy synthetic trajectory datasets (e.g., Multifurc1, Cyclic1, Disconn1, Conn1; ~1000 cells, ~1000 features) with gold-standard milestone_network, pseudotime, and cell-fate labels
- First 10 principal components of each synthetic dataset (e.g., stored in adata.obsm['X_pca'])
- Inferred trajectories from each TI method (VIA, Palantir, PAGA, Slingshot, STREAM, Monocle3)

## Outputs

- Per-dataset metric scores for all five dynverse metrics (GED, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, F1 cell fate) per method
- Composite accuracy table ranking VIA against all baselines (e.g., saved as dyntoy_ti_benchmark_metrics.csv)
- Validated recovery of terminal states (e.g., the elusive M2 terminal state on the four-leaf multifurcation dataset)

## How to apply

Generate nine dyntoy synthetic datasets (e.g., Multifurc1, Cyclic1, Disconn1, Conn1) with ~1000 cells each using dynverse/dyntoy, retaining the gold-standard topology, pseudotime, and cell-fate labels. Reduce each dataset to its first 10 PCs, then run the candidate method (e.g., VIA via via.via_wrapper with knn=20 and a fixed random_seed for connected topologies, or via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True for Disconn1) alongside baselines (Monocle3, Slingshot, STREAM, PAGA, Palantir) under default or benchmark-prescribed settings. Score every inferred trajectory against the dyntoy gold standard with the five dynverse metrics: graph edit distance (GED), F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate. Aggregate per-dataset metric means into a composite accuracy table ranking the methods. The rationale is that dyntoy data are simple enough for rapid prototyping yet can simulate complex trajectory models such as large trees, convergences, and loops.

## Related tools

- **VIA** (Candidate TI method benchmarked on the nine dyntoy datasets; run via via.via_wrapper (knn=20, fixed seed) for connected topologies and via.via_wrapper_disconnected (root=[1,1], preserve_disconnected=True) for disconnected ones) — https://github.com/ShobiStassen/VIA
- **dynverse/dyntoy** (Generates the synthetic single-cell trajectory datasets with gold-standard topology, pseudotime, and cell-fate labels used as benchmark ground truth) — https://github.com/dynverse/dyntoy
- **Palantir** (Baseline TI method scored against the dyntoy gold standard with the five dynverse metrics)
- **PAGA** (Baseline TI method scored against the dyntoy gold standard with the five dynverse metrics)
- **Slingshot** (Baseline TI method scored against the dyntoy gold standard with the five dynverse metrics)
- **STREAM** (Baseline TI method scored against the dyntoy gold standard with the five dynverse metrics)
- **Monocle3** (Baseline TI method scored against the dyntoy gold standard with the five dynverse metrics)
- **PARC** (Clustering algorithm underlying VIA's cluster-graph construction during the benchmark runs) — https://github.com/ShobiStassen/VIA

## Examples

```
library(dyntoy); dataset <- generate_dataset(model = model_bifurcating(), num_cells = 1000, num_features = 1000); dataset$milestone_network
```

## Evaluation signals

- VIA detects the elusive M2 terminal state on the four-leaf multifurcation dataset while Palantir, PAGA, Slingshot, STREAM, and Monocle3 all merge it with the M8 lineage
- VIA correctly infers the cyclic and disconnected topologies that tree-based baselines miss
- VIA attains the best composite accuracy when per-dataset metric means are aggregated across the five dynverse metrics
- All five dynverse metrics (GED, F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, F1 cell fate) are computed for every method-dataset pair before ranking
- Each dyntoy dataset contains ~1000 cells and retains its gold-standard milestone_network, pseudotime, and cell-fate labels for scoring

## Limitations

- dyntoy's generation model is very simplistic and far from realistic; for more realistic synthetic data, the README recommends the dyngen package
- dyntoy data are relatively easy and best suited to quickly test and prototype TI methods, not to definitively rank methods on realistic data
- No software version numbers for VIA or the compared methods are stated in the discussion, complicating exact reproducibility

## Evidence

- [readme] Even though the model to generate the data is very simplistic (and far from realistic), it can simulate very complex trajectory models, such as large trees, convergences and loops.: [evidence span withheld — anchor status: external_doc]
- [readme] As the data is relatively easy, it can be used to quickly test and prototype a TI method.: [evidence span withheld — anchor status: not_found]
- [methods] via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [intro] current trajectory inference (TI) methods predominantly work well on tree-like trajectories (e.g., Slingshot and STREAM), but lack the generalizability to infer disconnected, cyclic or hybrid: "current trajectory inference (TI) methods predominantly work well on tree-like trajectories (e.g., Slingshot and STREAM), but lack the generalizability to infer disconnected, cyclic or hybrid"
- [readme] however, for more realistic synthetic data, check out our [dyngen package](https://github.com/dynverse/dyngen).: "However, for more realistic synthetic data, check out our [dyngen
package](https://github.com/dynverse/dyngen)."
