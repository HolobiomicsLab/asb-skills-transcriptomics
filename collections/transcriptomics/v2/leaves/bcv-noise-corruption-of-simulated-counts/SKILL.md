---
name: bcv-noise-corruption-of-simulated-counts
description: Use when benchmarking feature selection or trajectory inference methods
  under quantitatively controlled signal-to-noise regimes, requiring simulated scRNA-seq
  count data with known ground-truth genes and labels.
when_to_use_negative:
- Real experimental scRNA-seq data where noise is uncontrolled and no simulation ground
  truth exists — use empirical evaluation instead
- Benchmarking tasks requiring dropout or library-size noise variation, since these
  simulations set dropout = None and fix library-size parameters while varying only
  BCV
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: Splatter
  role: R package used via the paths function to simulate linear-trajectory scRNA-seq
    counts with tunable BCV noise
- name: scprep (SplatSimulate)
  role: Python wrapper through which SplatSimulate is called to generate the simulated
    datasets
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: Feature selection method evaluated on the BCV-varied simulations
  repo: https://github.com/jranek/delve
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:486c501b5617e0d7d979185528a12e815e0c256f179e8250fd29073032f572d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Python
  - Scanpy
  - Splatter
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# bcv-noise-corruption-of-simulated-counts

## Summary

Generate controlled-noise single-cell RNA-seq simulations by varying the Splatter biological coefficient of variation (BCV = 0.1, 0.25, 0.5) across linear-trajectory datasets with known ground truth, enabling noise-stratified benchmarking of feature selection methods.

## When to use

Use when benchmarking feature selection or trajectory inference methods under quantitatively controlled signal-to-noise regimes, requiring simulated scRNA-seq count data with known ground-truth genes and labels.

## When NOT to use

- Real experimental scRNA-seq data where noise is uncontrolled and no simulation ground truth exists — use empirical evaluation instead
- Benchmarking tasks requiring dropout or library-size noise variation, since these simulations set dropout = None and fix library-size parameters while varying only BCV

## Inputs

- Splatter simulation parameters (mean_rate = 0.0173, mean_shape = 0.54, lib_loc = 12.6, lib_scale = 0.423, bcv_df = 90.2, dropout = None)
- Noise-level specification (BCV = 0.1, 0.25, 0.5)
- Simulation size specification (1500 cells, 500 genes, 6 clusters, 30 datasets per condition)

## Outputs

- Simulated scRNA-seq count matrices per BCV condition with ground-truth cluster labels, per-cluster/path differentially expressed genes, and latent step (pseudotime) annotations
- Aggregated median metric scores per noise condition per method (CSV results table)

## How to apply

Generate 30 linear-trajectory datasets per noise condition (1500 cells, 500 genes, 6 clusters) using the splatter paths function via the scprep SplatSimulate wrapper with estimated parameters (mean_rate = 0.0173, mean_shape = 0.54, lib_loc = 12.6, lib_scale = 0.423, bcv_df = 90.2, dropout = None). Vary the biological coefficient of variation across BCV = 0.1 (high SNR), 0.25 (medium), and 0.5 (low SNR) to induce graded noise corruption of the simulated counts while retaining the ground-truth differentially expressed genes per cluster/path and the latent step vector. This graded BCV design is what lets downstream evaluation report noise-stratified metrics, e.g. DELVE's median k-NN cell type classification accuracy of 0.937 (high), 0.882 (medium), and 0.734 (low SNR). Aggregate median metric scores across the datasets per noise condition for comparison across methods.

## Related tools

- **Splatter** (R package used via the paths function to simulate linear-trajectory scRNA-seq counts with tunable BCV noise)
- **scprep (SplatSimulate)** (Python wrapper through which SplatSimulate is called to generate the simulated datasets) — https://github.com/jranek/delve_benchmark
- **DELVE** (Feature selection method evaluated on the BCV-varied simulations) — https://github.com/jranek/delve

## Examples

```
sim = scprep.run.SplatSimulate(method='paths', n_cells=1500, n_genes=500, group_prob=..., mean_rate=0.0173, mean_shape=0.54, lib_loc=12.6, lib_scale=0.423, bcv_df=90.2, bcv_common=0.5, dropout=None)
```

## Evaluation signals

- Each simulated dataset has the specified dimensions: 1500 cells x 500 genes with 6 clusters, and 30 datasets are produced per BCV condition
- Ground-truth annotations are present and complete: per-cluster/path differentially expressed genes, cluster labels, and the latent step vector for pseudotime correlation
- Only the biological coefficient of variation changes between conditions (0.1, 0.25, 0.5); all other Splatter parameters remain fixed at the estimated values
- Expected SNR-ordered accuracy pattern: median k-NN cell type classification accuracy decreases with increasing BCV (e.g. 0.937 high, 0.882 medium, 0.734 low for DELVE)

## Limitations

- Dropout is disabled (dropout = None), so this noise model captures BCV-driven low signal-to-noise but not zero-inflation noise
- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis

## Evidence

- [readme] varying the biological coefficient of variation across BCV = 0.1, 0.25, 0.5: [evidence span withheld — anchor status: not_found]
- [readme] DELVE benchmark simulation parameters: [evidence span withheld — anchor status: not_found]
- [results] noise-stratified accuracy finding: [evidence span withheld — anchor status: not_found]
- [discussion] benchmarking motivation: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data"
