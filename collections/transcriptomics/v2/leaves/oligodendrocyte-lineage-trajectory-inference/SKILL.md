---
name: oligodendrocyte-lineage-trajectory-inference
description: Use when an AnnData object with spliced/unspliced counts and cell-type
  cluster labels covers a linear oligodendrocyte differentiation (e.g., P20 pons)
  and you must recover ordering and quantify directional correctness via CBDir.
when_to_use_negative:
- Datasets with cycling progenitors or multiple branching lineages (e.g., E15.5 retina)
  where the independent mode (FIT_OPTION='2') with gene-specific time is more appropriate
  than the unified-time mode
- Input lacking unspliced counts — RNA velocity estimation cannot run without spliced/unspliced
  layers
- Trajectories with no ground-truth cluster ordering, since CBDir evaluation via utv.evaluate
  requires cluster_edges
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Primary velocity estimator run in unified-time mode (FIT_OPTION='1') to recover
    the continuous OPC→COP→NFOL→MFOL path on the pons dataset
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Baseline method (stochastic and dynamical modes) whose trajectories and CBDir
    scores are compared against UniTVelo on the same dataset
- name: Python 3
  role: Implementation environment for UniTVelo (with TensorFlow 2) and the benchmarking
    scripts
  repo: https://github.com/StatBiomed/UniTVelo
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:7b5e6323138a51e789eb63b66f1a56710ea5902b6ed3db03d11cb71814676245
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# oligodendrocyte-lineage-trajectory-inference

## Summary

Recovers the continuous OPC→COP→NFOL→MFOL oligodendrocyte maturation path in adolescent (P20) mouse hindbrain (pons) scRNA-seq data using UniTVelo's unified-time mode, and benchmarks it against scVelo (stochastic/dynamical) with the Cross-Boundary Direction Correctness (CBDir) metric.

## When to use

Use when an AnnData object with spliced/unspliced counts and cell-type cluster labels covers a linear oligodendrocyte differentiation (e.g., P20 pons) and you must recover ordering and quantify directional correctness via CBDir.

## When NOT to use

- Datasets with cycling progenitors or multiple branching lineages (e.g., E15.5 retina) where the independent mode (FIT_OPTION='2') with gene-specific time is more appropriate than the unified-time mode
- Input lacking unspliced counts — RNA velocity estimation cannot run without spliced/unspliced layers
- Trajectories with no ground-truth cluster ordering, since CBDir evaluation via utv.evaluate requires cluster_edges

## Inputs

- AnnData (.h5ad) of P20 mouse hindbrain (pons) with spliced and unspliced count layers
- Cell-type cluster annotation column in adata.obs (e.g., OPC, COP, NFOL, MFOL)
- Ground-truth cluster_edges list defining OPC→COP→NFOL→MFOL transitions for CBDir

## Outputs

- Per-gene/per-cell RNA velocity estimates and a unified latent time stored in the AnnData
- Velocity embedding stream plots (scv.pl.velocity_embedding_stream, dpi=100, colored by cell-type label)
- CBDir scores table comparing UniTVelo vs scVelo (stochastic, dynamical) via utv.evaluate

## How to apply

Load the P20 hindbrain (pons) AnnData with spliced/unspliced layers and the cluster-label column in adata.obs. Configure UniTVelo via utv.config.Configuration in unified-time mode (FIT_OPTION='1') with R2_ADJUST=True, IROOT=None, and the GPU index, then fit with utv.run_model(path_to_adata, label, config_file=velo) to estimate gene velocities and a single transcriptome-wide latent time. Fit scVelo (stochastic and dynamical modes) on the same data as baseline and render scv.pl.velocity_embedding_stream plots (dpi=100, colored by the label column) to visually check whether the full OPC→COP→NFOL→MFOL ordering is recovered. Compute Cross-Boundary Direction Correctness with utv.evaluate(adata, cluster_edges, label, 'velocity') over the ground-truth OPC→COP→NFOL→MFOL cluster edges and tabulate CBDir for both methods; accept if UniTVelo's stream shows the continuous maturation path where scVelo does not, with CBDir consistent with the paper (0.609 vs 0.347 and 0.074, Supplementary Fig. S7c). The unified latent time is the rationale: it resolves directionality discrepancies between genes, which is critical for a mostly linear maturation path that scVelo's per-gene estimates fail to order correctly.

## Related tools

- **UniTVelo** (Primary velocity estimator run in unified-time mode (FIT_OPTION='1') to recover the continuous OPC→COP→NFOL→MFOL path on the pons dataset) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Baseline method (stochastic and dynamical modes) whose trajectories and CBDir scores are compared against UniTVelo on the same dataset)
- **Python 3** (Implementation environment for UniTVelo (with TensorFlow 2) and the benchmarking scripts) — https://github.com/StatBiomed/UniTVelo

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title=''); metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- UniTVelo's velocity stream shows the continuous OPC→COP→NFOL→MFOL ordering while scVelo fails to recover the full differentiation trajectory
- UniTVelo CBDir ≈ 0.609 vs scVelo stochastic ≈ 0.347 and dynamical ≈ 0.074 (paper's Table 1 / Supplementary Fig. S7c)
- utv.evaluate returns the CBDir metric for 'velocity' without errors using the specified cluster_edges
- Velocity embedding stream renders with dpi=100 colored by the cell-type label column, and latent time increases along OPC→MFOL

## Limitations

- scVelo fails to recover the full differentiation trajectory on this dataset, so baseline comparison may show near-random directionality (dynamical CBDir 0.074)
- UniTVelo's unified-time mode is best for linear/ordered paths; complex multi-branch or cell-cycle datasets need the independent mode (FIT_OPTION='2')
- GPU memory constrains scale — the paper notes the sub-sampling strategy for datasets larger than ~50,000 cells on a 12GB GPU (NVIDIA GeForce RTX 2080 Ti)
- No unsupervised method exists for selecting dynamically informed genes for projecting cell transitions; unspliced over-counting can affect cell transition metrics

## Evidence

- [methods] Unified-time mode with CBDir comparison on pons: "Run model (utv.run_model) (label refers to column name in adata.obs specifying celltypes)::
    adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [intro] Unified latent time rationale: "we also introduce a unified latent time across the whole transcriptome, which can resolve the discrepancy of directionality between genes"
- [readme] Configuration parameters from README: "velo.R2_ADJUST = True 
velo.IROOT = None
velo.FIT_OPTION = '1'
velo.GPU = 0"
- [readme] CBDir evaluation: "metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
- [results] Task finding on pons trajectory and CBDir scores: [evidence span withheld — anchor status: not_found]
- [discussion] GPU memory limitation: "larger than 50,000 cells for a GPU with 12GB memory"
