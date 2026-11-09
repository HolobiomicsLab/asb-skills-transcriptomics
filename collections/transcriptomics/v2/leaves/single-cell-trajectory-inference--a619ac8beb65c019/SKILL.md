---
name: single-cell-trajectory-inference
description: Use when scRNA-seq (or CyTOF/scATAC/imaging) data must be ordered along
  pseudotime and cell fates/terminal states auto-detected — e.g. rare delta cells
  in E15.5 pancreatic endocrinogenesis or dendritic subtypes in CD34+ hematopoiesis.
when_to_use_negative:
- The topology is already known and user-defined terminal/fate assignment is desired
  — VIA's value here is automatic, unsupervised fate detection without manual cell-fate
  assignment
- The input is not single-cell level data (e.g. bulk time-course without per-cell
  resolution), since VIA's cluster-graph and random-walk machinery operate on single-cell
  graphs
- Extensive pre-dimensionality reduction or subsampling is planned that would destroy
  global connectivity — VIA is designed to scale to >10^6 cells without it
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: 'Core trajectory-inference algorithm: pseudotime computation, automated terminal-state
    detection, and lineage branch probabilities via lazy-teleporting random walks
    with MCMC'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Underlying single-cell clustering that builds the cluster-graph; edge pruning
    controlled by jac_std_global and too_big_factor for cluster granularity
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Provides the low-dimensional embedding used for single-cell level visualization
    of the inferred trajectory
- name: scanpy
  role: Assembles the AnnData object and stores PCA in adata.obsm['X_pca'] as input
    to VIA
- name: scVelo
  role: Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used for
    lineage detection (scv.datasets)
- name: dyntoy
  role: Simulated toy trajectory datasets (e.g. 4-leaf multifurcation, disconnected)
    for testing and prototyping the TI method
  repo: dynverse/dyntoy
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:db9940a410d1f040ac039de3f8ca51c1d173544a1fbab66e55edd62c1ab1f607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - dyntoy
  - pandas
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-trajectory-inference

## Summary

Reconstruct single-cell developmental trajectories and automatically detect terminal states and rare lineages using VIA, a graph-based trajectory-inference (TI) algorithm that clusters cells with PARC, computes pseudotime, and infers lineage probabilities via lazy-teleporting random walks with MCMC refinement.

## When to use

Use when scRNA-seq (or CyTOF/scATAC/imaging) data must be ordered along pseudotime and cell fates/terminal states auto-detected — e.g. rare delta cells in E15.5 pancreatic endocrinogenesis or dendritic subtypes in CD34+ hematopoiesis.

## When NOT to use

- The topology is already known and user-defined terminal/fate assignment is desired — VIA's value here is automatic, unsupervised fate detection without manual cell-fate assignment
- The input is not single-cell level data (e.g. bulk time-course without per-cell resolution), since VIA's cluster-graph and random-walk machinery operate on single-cell graphs
- Extensive pre-dimensionality reduction or subsampling is planned that would destroy global connectivity — VIA is designed to scale to >10^6 cells without it

## Inputs

- AnnData object (or equivalent matrix) with scRNA-seq counts/expression and PCA stored in adata.obsm['X_pca']
- Optional cell-type labels (true_label) and cluster annotation vector for coloring/plots
- Low-dimensional embedding (UMAP or PHATE) for single-cell visualization
- Marker gene list (e.g. Sst, Hhex, Cd24a, Ins1, Ins2, Gcg; IRF8, CSF1R, CD123)
- User-specified root/start cell via root_user (e.g. root=[42])

## Outputs

- Pseudotime values per cell
- via_object.terminal_clusters — auto-detected terminal state cluster IDs
- via_object.single_cell_bp — single-cell branch/lineage probabilities per terminal state
- Cluster-graph trajectory and edge-bundled visualizations on the UMAP/PHATE embedding
- Gene-expression trends along pseudotime/lineages (e.g. marker dynamics plots, CSV tables of F1 scores and marker-pseudotime correlations)

## How to apply

Load the expression data as an AnnData object with a PCA representation stored in adata.obsm['X_pca'] and compute a UMAP embedding (e.g. umap-learn with random_state=42, n_neighbors=15) for single-cell visualization. Run VIA through its wrapper — via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42]) — setting root_user to an early start cell and, for disconnected topologies, using via_wrapper_disconnected with preserve_disconnected=True. Leave terminal-state detection automatic: the default pseudotime_threshold_TS=30 percentile criterion qualifies nodes as terminal states, and too_big_factor=0.4 splits oversized PARC clusters for finer granularity; tune jac_std_global (PARC clustering edge pruning) and knn (HNSWlib graph neighbors) as key sensitivity knobs. Extract via_object.terminal_clusters and via_object.single_cell_bp branch probabilities (estimated with num_sim_branch_probability=500 MCMCs per terminal state) to enumerate fates, then validate each predicted fate by marker-gene expression (e.g. Sst/Hhex/Cd24a for delta cells; IRF8/CSF1R/CD123 for DC subtypes) and marker trends along pseudotime via VIA's gene-expression plotting.

## Related tools

- **VIA** (Core trajectory-inference algorithm: pseudotime computation, automated terminal-state detection, and lineage branch probabilities via lazy-teleporting random walks with MCMC) — https://github.com/ShobiStassen/VIA
- **PARC** (Underlying single-cell clustering that builds the cluster-graph; edge pruning controlled by jac_std_global and too_big_factor for cluster granularity) — https://github.com/ShobiStassen/VIA
- **UMAP** (Provides the low-dimensional embedding used for single-cell level visualization of the inferred trajectory)
- **scanpy** (Assembles the AnnData object and stores PCA in adata.obsm['X_pca'] as input to VIA)
- **scVelo** (Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used for lineage detection (scv.datasets))
- **dyntoy** (Simulated toy trajectory datasets (e.g. 4-leaf multifurcation, disconnected) for testing and prototyping the TI method) — dynverse/dyntoy

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42])
```

## Evaluation signals

- The run auto-detects the expected number of terminal clusters without user-supplied terminal states (e.g. exactly five for the E15.5 endocrine fates: Alpha, Beta-1, Beta-2, Delta, Epsilon)
- Rare lineages are recovered: the Delta terminal state matches the ~3% population with exclusively elevated Hhex, Sst, and Cd24a
- Pseudotime ordering is biologically consistent, e.g. Beta-2 (Ins2+Ins1-low immature) precedes Beta-1 (Ins1+Ins2+ mature) in pseudotime
- Terminal-state detection F1-scores per lineage remain robust when sweeping knn (around the default 30) and ncomps (~10-20) at fixed jac_std_global=0.15
- Marker trends along pseudotime validate fates: IRF8 elevated in both DC lineages, CSF1R specific to cDC, CD123 elevated in pDCs

## Limitations

- The 'next fastest method' runtime comparator, software versions, and dataset accession identifiers for validation datasets (hematopoiesis, pancreatic development, MOCA) are not stated, complicating exact reproduction
- Results such as stress-test parameter ranges and quantitative accuracy metrics defer to supplementary figures rather than the main text
- Methods requiring UMAP/t-SNE before parsing large atlases (e.g. MOCA) are highly susceptible to user-defined input parameters, so embedding choice can affect visualization
