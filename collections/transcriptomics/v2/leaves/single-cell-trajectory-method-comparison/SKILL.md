---
name: single-cell-trajectory-method-comparison
description: Use when comparing RNA-velocity-based trajectory inference (Cytopath)
  against topology-constrained baselines on a scRNA-seq dataset with pre-computed
  velocity and known root/terminal cell states and a known cell type ordering.
when_to_use_negative:
- The underlying RNA velocity estimation itself is faulty — the authors state trajectory
  inference with Cytopath is not meaningful in that case (e.g. spurious root/terminal
  inference on the erythroid gastrulation dataset)
- Root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend performing trajectory inference with Cytopath in this situation
- The differentiation topology violates the method's assumptions, e.g. tree-constrained
  methods like Slingshot on convergent/circular topologies such as the neonatal mouse
  inner ear dataset
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Cytopath
  role: Primary simulation-based trajectory inference method run 10 times under root/terminal
    supervision
  repo: github.com/aron0093/cytopath
- name: Slingshot
  role: Baseline trajectory inference method for tree-like topology, run with identical
    root/terminal states
- name: PAGA
  role: Baseline graph abstraction method with directionality enabled by velocity
    pseudotime
- name: Monocle3
  role: 'Baseline trajectory inference method (documented failure mode: disconnected
    graph)'
- name: VeTra
  role: Baseline RNA-velocity-aware trajectory inference method
- name: Cellpath
  role: Baseline RNA-velocity-aware trajectory inference method
- name: scvelo
  role: Loads the hippocampus dataset with pre-computed RNA velocity and supplies
    latent time / velocity pseudotime for PAGA directionality
- name: dynmethods
  role: dynverse collection of 56 wrapped trajectory inference methods for running/comparing
    TI methods in a common trajectory model via dynwrap
  repo: https://github.com/dynverse/dynmethods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:eb9c0b69671adec87675f34d0cf645936d46d15896de566b836a837950c480c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellpath
  - Cytopath
  - Monocle3
  - PAGA
  - Slingshot
  - VeTra
  - dynmethods
  - scvelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-trajectory-method-comparison

## Summary

Benchmark Cytopath against baseline trajectory inference methods (Slingshot, PAGA with velocity-pseudotime directionality, Monocle3, VeTra, Cellpath) on the 18,140-cell developing mouse hippocampus dataset, quantifying pseudotime quality via Spearman correlation with known cell type ordering across repeated runs.

## When to use

Use when comparing RNA-velocity-based trajectory inference (Cytopath) against topology-constrained baselines on a scRNA-seq dataset with pre-computed velocity and known root/terminal cell states and a known cell type ordering.

## When NOT to use

- The underlying RNA velocity estimation itself is faulty — the authors state trajectory inference with Cytopath is not meaningful in that case (e.g. spurious root/terminal inference on the erythroid gastrulation dataset)
- Root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend performing trajectory inference with Cytopath in this situation
- The differentiation topology violates the method's assumptions, e.g. tree-constrained methods like Slingshot on convergent/circular topologies such as the neonatal mouse inner ear dataset

## Inputs

- Developing mouse hippocampus scRNA-seq dataset (18,140 cells) with pre-computed RNA velocity (scvelo/anndata format, loom-derived spliced/unspliced counts)
- Supplied root and terminal cell states (labels or probability annotations)
- Known ordering of cell types per lineage for correlation ground truth
- Clustering of cells (or Cytopath computes Louvain clustering internally)

## Outputs

- Per-method, per-lineage, per-run Spearman correlation table (e.g. hippocampus_pseudotime_correlations.csv)
- Inferred pseudotime per trajectory for Cytopath (per initialisation) and each baseline method
- Recorded baseline failure modes (e.g. Monocle3 disconnected graph, reversed PAGA directionality)
- Robustness assessment of Cytopath correlations across the 10 independent initialisations

## How to apply

Load the developing mouse hippocampus dataset (18,140 cells) with pre-computed RNA velocity using scvelo/anndata, supplying the known root and terminal cell states. Run Cytopath over 10 independent initialisations, inferring pseudotime for each trajectory under identical root/terminal supervision; Cytopath defines a Markov chain model, simulates an ensemble of differentiation trajectories, aligns simulations terminating at a common terminal state with dynamic time warping, and assigns cells to trajectory steps by an alignment score of static and velocity profiles. Run the five baselines — Slingshot, PAGA with velocity-pseudotime-based directionality, Monocle3, VeTra, and Cellpath — with the identical root and terminal states. Compute the Spearman correlation of each method's pseudotime with the known ordering of cell types per lineage (scipy.stats.spearmanr), assess robustness of Cytopath correlations across the 10 runs, and record baseline failure modes such as Monocle3's disconnected-graph failure and reversed PAGA directionality. Write per-method, per-lineage, per-run correlations to a CSV table.

## Related tools

- **Cytopath** (Primary simulation-based trajectory inference method run 10 times under root/terminal supervision) — github.com/aron0093/cytopath
- **Slingshot** (Baseline trajectory inference method for tree-like topology, run with identical root/terminal states)
- **PAGA** (Baseline graph abstraction method with directionality enabled by velocity pseudotime)
- **Monocle3** (Baseline trajectory inference method (documented failure mode: disconnected graph))
- **VeTra** (Baseline RNA-velocity-aware trajectory inference method)
- **Cellpath** (Baseline RNA-velocity-aware trajectory inference method)
- **scvelo** (Loads the hippocampus dataset with pre-computed RNA velocity and supplies latent time / velocity pseudotime for PAGA directionality)
- **dynmethods** (dynverse collection of 56 wrapped trajectory inference methods for running/comparing TI methods in a common trajectory model via dynwrap) — https://github.com/dynverse/dynmethods

## Examples

```
import scvelo as scv; adata = scv.read('hippocampus.h5ad'); import cytopath; cytopath.trajectory(adata, clusters='clusters', source='root', target='terminal', n_jobs=8)
```

## Evaluation signals

- Spearman correlation between Cytopath-inferred pseudotime per trajectory and the known cell type ordering is high (Figure 2D)
- Cytopath correlations are robust across multiple independent runs/initialisations (Figure 2E)
- Baselines run with identical root/terminal supervision should reproduce expected orderings; deviations such as reversed PAGA directionality or Monocle3 disconnected-graph failure are expected and must be recorded
- Output CSV contains one row per method x lineage x run with valid Spearman correlation values in [-1, 1]
- Cytopath pseudotime was reported robust to root and terminal probability thresholds within a range of [0.85, 1]

## Limitations

- Robustness of Cytopath pseudotime to root and terminal probability thresholds is reported only as 'data not shown' for the [0.85, 1] range
- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene)
- Incorrect RNA velocity estimation leads to spurious inference of root and terminal cell states (e.g. erythroid gastrulation dataset; fuzzy root cell probability in hepatocyte zonation)
- The correctness of RNA velocity estimation cannot be verified in every scenario, even when marker-based information is available

## Evidence

- [results] hippocampus robust pseudotime: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [results] dataset description: "we applied Cytopath and baseline methods to the developing mouse hippocampus dataset, which was first used to demonstrate RNA velocity of single cells. This dataset is composed of 18,140 cells."
- [results] baselines: "We also include a comparison with Monocle316 as well as two approaches accounting for RNA velocity information: VeTra11 and Cellpath.12"
- [abstract] cytopath algorithm: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [results] alignment method: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [discussion] scope caveat: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
- [discussion] velocity caveat: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
- [readme] README scope: "Cytopath can infer trajectories with or without root/terminal state supervision. No topological constraints (e.g. a tree structure) are placed on the inference as each trajectory is modelled"
- [readme] dynmethods comparison framing: [evidence span withheld — anchor status: not_found]
