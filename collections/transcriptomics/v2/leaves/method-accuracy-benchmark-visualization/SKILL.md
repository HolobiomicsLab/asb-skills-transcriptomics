---
name: method-accuracy-benchmark-visualization
description: Use when you must compare the robustness of two cell-cycle/pseudo-time
  methods by down-sampling an annotated scRNA-seq matrix to fewer cells (e.g. n=10,
  n=30) or fewer genes and plotting median accuracy against sample size.
when_to_use_negative:
- The question is method development or pseudo-time inference quality itself, not
  a head-to-head accuracy robustness comparison under down-sampling
- The dataset lacks annotated cell-cycle stage labels, so stratified cell down-sampling
  and stage-prediction accuracy cannot be computed
- Both candidate methods require full-scale data (e.g. reCAT fails below 5000 genes),
  making down-sampled comparisons uninformative for that method
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cyclum
  role: Sinusoidal autoencoder projecting expression onto a circular manifold; cyclum.tuning.CyclumAutoTune
    fixes the number of linear dimensions; the comparator expected to stay robust
    (median accuracy 0.70-0.75) under down-sampling
  repo: github.com/KChen-lab/cyclum
- name: reCAT
  role: Gaussian mixture model-based pseudo-time reconstruction run on identical down-sampled
    matrices as the model-free comparator whose accuracy degrades with fewer cells
    and fails below 5000 genes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:0111d2fb81b2024fb4b1a7d7b42757ea6d22b9edc4d8ec5b1fce097dd8c9dc76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - reCAT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# method-accuracy-benchmark-visualization

## Summary

Benchmark and visualize classification accuracy of competing methods (here Cyclum vs. reCAT on mESC scRNA-seq) under controlled down-sampling of cells and genes, producing accuracy tables and accuracy-vs-sample-size curves that expose robustness differences.

## When to use

Use when you must compare the robustness of two cell-cycle/pseudo-time methods by down-sampling an annotated scRNA-seq matrix to fewer cells (e.g. n=10, n=30) or fewer genes and plotting median accuracy against sample size.

## When NOT to use

- The question is method development or pseudo-time inference quality itself, not a head-to-head accuracy robustness comparison under down-sampling
- The dataset lacks annotated cell-cycle stage labels, so stratified cell down-sampling and stage-prediction accuracy cannot be computed
- Both candidate methods require full-scale data (e.g. reCAT fails below 5000 genes), making down-sampled comparisons uninformative for that method

## Inputs

- mESC scRNA-seq expression matrix (genes x cells) with annotated cell-cycle stage labels
- Down-sampling design parameters: stratified cell counts (n=10, n=30) and decreasing gene-count series

## Outputs

- accuracy_comparison.csv — per-condition cell-cycle classification accuracy for Cyclum and reCAT
- fig2cd_accuracy_robustness.png — accuracy curves versus number of cells and versus number of genes

## How to apply

Load the mESC scRNA-seq expression matrix and preprocess it with a log2(x+1) transform, then scale to variance = 1 and center (mean = 1) as recommended in the Cyclum FAQ. Generate replicate down-sampled datasets by stratifying cells on annotated stage (n=10 and n=30 cells) and uniformly sampling genes at a decreasing series of gene counts. Run Cyclum (sinusoidal autoencoder projecting expression onto a circular manifold, using cyclum.tuning.CyclumAutoTune to fix the number of linear dimensions) and reCAT (Gaussian mixture model-based pseudo-time reconstruction) on the identical down-sampled matrices. Compute cell-cycle stage prediction accuracy per condition, save the table to accuracy_comparison.csv, and plot accuracy versus number of cells and versus number of genes (fig2cd_accuracy_robustness.png). Accept the reproduction when Cyclum's median accuracy stays within the 0.70-0.75 band across conditions while reCAT degrades and fails below 5000 genes.

## Related tools

- **Cyclum** (Sinusoidal autoencoder projecting expression onto a circular manifold; cyclum.tuning.CyclumAutoTune fixes the number of linear dimensions; the comparator expected to stay robust (median accuracy 0.70-0.75) under down-sampling) — github.com/KChen-lab/cyclum
- **reCAT** (Gaussian mixture model-based pseudo-time reconstruction run on identical down-sampled matrices as the model-free comparator whose accuracy degrades with fewer cells and fails below 5000 genes)

## Examples

```
import cyclum.tuning; tuner = cyclum.tuning.CyclumAutoTune(n_component=..., max_length=...); model = tuner.train(dis_epoch=50, con_epoch=50); pseudotime = model.predict(expression_matrix)
```

## Evaluation signals

- Cyclum's median cell-cycle classification accuracy remains within the 0.70-0.75 band across all cell-count and gene-count down-sampling conditions
- reCAT's median accuracy becomes substantially worse as the number of cells decreases
- reCAT fails below 5000 genes in the gene-count down-sampling series
- accuracy_comparison.csv contains one accuracy value per method per down-sampling condition, matching the plotted curves in fig2cd_accuracy_robustness.png
- Preprocessed matrix confirmed as scaled (variance = 1) and centered (mean = 1) after the log2(x+1) transform

## Limitations

- reCAT, as a model-free GMM approach, cannot be evaluated below ~5000 genes, so the gene-count benchmark curve for reCAT terminates early
- Cyclum requires TensorFlow 1.x (tested with tensorflow 1.14.0, keras 2.2.4, numpy 1.16.5, python 3.7.4); newer dependency versions may break the pipeline
- Accuracy is measured against annotated stage labels; the benchmark is only meaningful where trustworthy stage annotations exist

## Evidence

- [results] Cyclum's median cell-cycle classification accuracy (between 0.7 and 0.75) remained largely invariant with the number of cells, while reCAT's median accuracy became substantially worse with fewer cells.: [evidence span withheld — anchor status: not_found]
- [methods] scaled (variance = 1), centered (mean = 1) log2(x+1) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes.: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [readme] We also implemented class `cyclum.tuning.CyclumAutoTune`, which automatically select the proper number of linear components to help find the "most circular" manifold.: [evidence span withheld — anchor status: external_doc]
- [intro] A recent method reCAT25 reconstructs cell-cycle pseudo-time using a Gaussian mixture model (GMM): "A recent method reCAT25 reconstructs cell-cycle pseudo-time using a Gaussian mixture model (GMM)"
