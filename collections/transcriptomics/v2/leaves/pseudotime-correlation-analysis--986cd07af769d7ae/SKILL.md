---
name: pseudotime-correlation-analysis
description: Use when benchmarking feature selection on simulated or labeled trajectory
  data where ground-truth latent progression (e.g., Splatter step vector, SymSim latent
  time) is available and pseudotime fidelity must be quantified.
when_to_use_negative:
- No ground-truth time/progression annotation exists, since Kendall correlation against
  latent time cannot be computed and the score is unverifiable
- The goal is de novo trajectory discovery on real data rather than benchmarking feature
  selection against a known trajectory
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3524
tools:
- name: Scanpy
  role: Provides the diffusion pseudotime (dpt) algorithm run on a k = 10 k-NN graph
    for trajectory inference and correlation evaluation
  repo: https://github.com/jranek/delve_benchmark
- name: delve_benchmark
  role: Implements the feature selection evaluation (fs class with select() and evaluate_select())
    whose outputs are scored by pseudotime correlation
  repo: https://github.com/jranek/delve_benchmark
- name: Splatter
  role: Simulates linear-trajectory datasets (via scprep SplatSimulate) supplying
    the ground-truth latent step vector used as the correlation reference
  repo: https://github.com/jranek/delve_benchmark
- name: SymSim
  role: Simulates five tree differentiation trajectories (10,000 cells, 20,000 genes)
    whose latent progression is the correlation ground truth
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:be6749cb7a824e156284b612728a5c7a6a0d9b54ceb76be76d8de3de9f9a75c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - Python
  - Scanpy
  - Splatter
  - SymSim
  - delve_benchmark
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pseudotime-correlation-analysis

## Summary

Evaluates whether selected genes preserve cellular progression by inferring diffusion pseudotime on a k-NN graph and computing Kendall rank correlation against ground-truth latent time in simulated single-cell RNA-seq benchmarks (Splatter, SymSim).

## When to use

Use when benchmarking feature selection on simulated or labeled trajectory data where ground-truth latent progression (e.g., Splatter step vector, SymSim latent time) is available and pseudotime fidelity must be quantified.

## When NOT to use

- No ground-truth time/progression annotation exists, since Kendall correlation against latent time cannot be computed and the score is unverifiable
- The goal is de novo trajectory discovery on real data rather than benchmarking feature selection against a known trajectory

## Inputs

- Cell-by-gene expression matrix restricted to selected features (or a preprocessed .h5ad object with PCA embedding)
- Ground-truth latent time/progression annotation per cell (e.g., Splatter latent step vector, SymSim latent progression)
- Ground-truth cell-type labels for companion classification evaluation
- Root-cell selection rule (smallest ground-truth time annotation)

## Outputs

- Diffusion pseudotime value per cell
- Kendall rank correlation coefficient between estimated pseudotime and ground-truth latent progression
- Aggregated median pseudotime correlation per feature-selection method × feature count (e.g., written to symsim_feature_selection_results.csv)

## How to apply

After feature selection, build a k-nearest neighbor graph (k = 10) on the selected-feature expression matrix and run the diffusion pseudotime algorithm (Scanpy dpt). Choose root cells as the 10 random cells with the smallest ground-truth time annotation, and average over roots to reduce seed sensitivity. Compute the Kendall rank correlation between the estimated diffusion pseudotime and the ground-truth latent progression vector; in the SymSim workflow, this is applied per dataset across feature thresholds p = 2000, 1000, 500, 250, 100, 50 and aggregated as the median correlation per method × α_mean × p combination alongside 3-fold k-NN (k = 3) cell-type classification accuracy. High correlation at small feature counts indicates that the selected genes recapitulate the cellular trajectory, the criterion on which DELVE, Laplacian score, and Hotspot outperformed variance-based methods in the SymSim benchmark.

## Related tools

- **Scanpy** (Provides the diffusion pseudotime (dpt) algorithm run on a k = 10 k-NN graph for trajectory inference and correlation evaluation) — https://github.com/jranek/delve_benchmark
- **delve_benchmark** (Implements the feature selection evaluation (fs class with select() and evaluate_select()) whose outputs are scored by pseudotime correlation) — https://github.com/jranek/delve_benchmark
- **Splatter** (Simulates linear-trajectory datasets (via scprep SplatSimulate) supplying the ground-truth latent step vector used as the correlation reference) — https://github.com/jranek/delve_benchmark
- **SymSim** (Simulates five tree differentiation trajectories (10,000 cells, 20,000 genes) whose latent progression is the correlation ground truth) — https://github.com/jranek/delve_benchmark

## Examples

```
import scanpy as sc; sc.tl.dpt(adata, n_dcs=15, n_branchings=1); tau = scipy.stats.kendalltau(adata.obs['dpt_pseudotime'], ground_truth_latent_time).correlation
```

## Evaluation signals

- Kendall rank correlation between diffusion pseudotime and ground-truth latent step/progression should be positive and high (similarity-based methods like DELVE achieved the most accurate progression estimation)
- Correlation should remain robust at small feature subsets (e.g., 100 genes) for DELVE/Laplacian score/Hotspot, while variance-based methods require ~2000 genes for similar performance
- Root cells must correspond to the cells with smallest ground-truth time annotation; averaging over 10 random root choices reduces seed sensitivity
- Median correlation aggregated across simulation replicates per method × α_mean × p should match the pattern of Supplementary Fig. 13 in a reproduction

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory; careful consideration is needed before interpreting pseudotime
- Results depend on root-cell choice and graph construction (k = 10); random root selection is averaged to mitigate seed sensitivity
- Applicable only where a trusted ground-truth progression exists (simulations with known latent time); uninterpretable on unannotated real data

## Evidence

- [other] Infer cellular progression with the diffusion pseudotime algorithm (Scanpy dpt on a k = 10 k-NN graph, 10 random root cells with the smallest ground truth time annotation) and compute the Kendall rank correlation between estimated pseudotime and the ground truth latent step vector.: [evidence span withheld — anchor status: not_found]
- [discussion] similarity-based feature selection methods achieved highest recovery of differentially expressed genes: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes"
- [other] Supplementary Fig. 13 pattern: DELVE/LS/Hotspot robust at 100 genes while variance-based methods need ~2000: [evidence span withheld — anchor status: not_found]
- [readme] README: fs class select() and evaluate_select() benchmark methods on trajectory preservation tasks: [evidence span withheld — anchor status: external_doc]
