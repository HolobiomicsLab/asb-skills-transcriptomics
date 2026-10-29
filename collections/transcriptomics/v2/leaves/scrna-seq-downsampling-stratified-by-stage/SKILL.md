---
name: scrna-seq-downsampling-stratified-by-stage
description: Use when benchmarking cell-cycle pseudo-time methods for robustness on
  scRNA-seq data, specifically to test whether model-based (Cyclum) vs model-free
  (reCAT) approaches degrade with fewer cells or genes.
when_to_use_negative:
- The goal is de novo cell-cycle pseudo-time inference on the full dataset rather
  than method robustness benchmarking
- The input data lacks per-cell annotated cell-cycle stage labels, making stage-stratified
  cell down-sampling impossible
- The dataset is not a periodic-process scRNA-seq problem (e.g., non-cyclic differentiation
  trajectories better suited to linear pseudo-time methods like Monocle)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cyclum
  role: Model-based sinusoidal autoencoder that projects down-sampled expression onto
    a circular manifold to infer pseudo-time; cyclum.tuning.CyclumAutoTune fixes the
    number of linear dimensions
  repo: github.com/KChen-lab/cyclum
- name: reCAT
  role: Model-free Gaussian mixture model comparator that reconstructs cell-cycle
    pseudo-time on the identical down-sampled matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:20e17783f4cea19c9d1fb7b212141f85f38aff7fc7afbe6605fbcf9c348c4265
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - reCAT
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrna-seq-downsampling-stratified-by-stage

## Summary

A robustness-evaluation skill that down-samples the mESC scRNA-seq expression matrix by cells (stratified by annotated cell-cycle stage, n=10 and n=30) and by genes (decreasing gene counts), then compares Cyclum's sinusoidal autoencoder against reCAT's GMM-based pseudo-time reconstruction to quantify how classification accuracy degrades.

## When to use

Use when benchmarking cell-cycle pseudo-time methods for robustness on scRNA-seq data, specifically to test whether model-based (Cyclum) vs model-free (reCAT) approaches degrade with fewer cells or genes.

## When NOT to use

- The goal is de novo cell-cycle pseudo-time inference on the full dataset rather than method robustness benchmarking
- The input data lacks per-cell annotated cell-cycle stage labels, making stage-stratified cell down-sampling impossible
- The dataset is not a periodic-process scRNA-seq problem (e.g., non-cyclic differentiation trajectories better suited to linear pseudo-time methods like Monocle)

## Inputs

- mESC scRNA-seq gene-by-cell expression matrix (from the cyclum2 repository notebook data)
- Per-cell annotated cell-cycle stage labels for stratified sampling

## Outputs

- accuracy_comparison.csv — cell-cycle stage classification accuracy per method per down-sampling condition
- fig2cd_accuracy_robustness.png — accuracy curves versus number of cells and versus number of genes
- Cyclum-inferred circular pseudo-time per down-sampled matrix
- reCAT-inferred pseudo-time per down-sampled matrix

## How to apply

Load the mESC scRNA-seq expression matrix from the cyclum2 repository notebook data and preprocess with log2(x+1) transform, scaling to variance = 1 and centering (mean = 1), as recommended in the Cyclum FAQ. Create stratified down-sampled replicates: sample cells to n=10 and n=30 per annotated stage, and uniformly sample genes at a decreasing series of gene counts. Run Cyclum (sinusoidal autoencoder projecting onto a circular manifold; use cyclum.tuning.CyclumAutoTune to fix the number of linear dimensions) and reCAT (Gaussian mixture model-based) on each identical down-sampled matrix. Compute cell-cycle stage prediction accuracy per condition, plot accuracy versus cell count and gene count, and save accuracy_comparison.csv and fig2cd_accuracy_robustness.png. Accept the reproduction if Cyclum's median accuracy stays within the 0.70–0.75 band while reCAT accuracy degrades and reCAT fails below 5000 genes.

## Related tools

- **Cyclum** (Model-based sinusoidal autoencoder that projects down-sampled expression onto a circular manifold to infer pseudo-time; cyclum.tuning.CyclumAutoTune fixes the number of linear dimensions) — github.com/KChen-lab/cyclum
- **reCAT** (Model-free Gaussian mixture model comparator that reconstructs cell-cycle pseudo-time on the identical down-sampled matrices)

## Examples

```
from cyclum.tuning import CyclumAutoTune; tunner = CyclumAutoTune(model, epochs=500); model = tunner.show_frequencies()
# then run Cyclum and reCAT on each down-sampled matrix and tabulate accuracies to accuracy_comparison.csv
```

## Evaluation signals

- Cyclum's median cell-cycle classification accuracy remains within the 0.70–0.75 band across all cell- and gene-down-sampling conditions
- reCAT's median accuracy degrades substantially as cell count decreases (e.g., at n=10 and n=30 stratified replicates)
- reCAT fails to produce usable results below 5000 genes while Cyclum still runs
- accuracy_comparison.csv contains an accuracy row per method for every down-sampling condition
- Plotted curves reproduce the invariant-Cyclum / degrading-reCAT pattern seen in fig2cd_accuracy_robustness.png

## Limitations

- reCAT, being model-free, is expected to fail below ~5000 genes, so conditions below that threshold only benchmark Cyclum's behavior
- Cyclum requires TensorFlow 1.x-era dependencies (tested with tensorflow 1.14.0, keras 2.2.4, numpy 1.16.5); newer versions may need pinning in setup.py
- The robustness conclusion is grounded in the mESC dataset with Cyclum-preprocessing conventions (log2(x+1), variance = 1 scaling, mean = 1 centering) and may not transfer to other tissues or preprocessing choices

## Evidence

- [methods] preprocess scRNA-seq data with scaled, centered, log2(x+1) transformation: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum robust to small sample sizes: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [discussion] Cyclum advantage over model-free reCAT: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [readme] CyclumAutoTune class: [evidence span withheld — anchor status: external_doc]
- [readme] Cyclum dependency versions: [evidence span withheld — anchor status: external_doc]
