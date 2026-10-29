---
name: scrna-seq-log2-variance-scaling-preprocessing
description: Use when preparing scRNA-seq count matrices (mESC, melanoma Mel78-88,
  hESC 10X Chromium) as input to Cyclum's circular-manifold autoencoder or comparators
  like reCAT, Seurat, or ccRemover.
when_to_use_negative:
- Input is already a normalized/feature-scaled expression matrix rather than raw counts
  — re-applying log2 and variance scaling would distort the data
- The downstream method requires its own normalization scheme (e.g., methods outside
  the Cyclum/reCAT comparison set that specify different transforms)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Primary consumer of the preprocessed matrix; its sinusoidal autoencoder requires
    scaled (variance=1), centered (mean=1) log2(x+1) data before projecting cells
    onto the circular manifold
  repo: https://github.com/KChen-lab/Cyclum
- name: reCAT
  role: Comparator method run on identically preprocessed matrices (e.g., down-sampled
    mESC) for fair accuracy comparison
- name: cyclum.tuning.CyclumAutoTune
  role: Applied after preprocessing to select the optimal number of linear dimensions
    for the circular manifold
  repo: https://github.com/lshh125/cyclum2
- name: ccRemover
  role: Linear cell-cycle regression baseline run on the same preprocessed matrices
    in correction comparisons
- name: Seurat
  role: Linear regression baseline consuming the identically preprocessed matrices
    for cell-cycle correction comparisons
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_008
  - task_004
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:d6ba790c7fbd7a92a35a618d53d4465a55ef865e1c21aa2e15c8ef70d0f7a66a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Seurat
  - TensorFlow
  - ccRemover
  - cyclum.tuning.CyclumAutoTune
  - reCAT
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrna-seq-log2-variance-scaling-preprocessing

## Summary

Preprocessing convention recommended by the Cyclum authors for scRNA-seq expression matrices: a log2(x+1) transform followed by scaling to variance = 1 and centering to mean = 1, so the sinusoidal autoencoder operates on comparable gene scales.

## When to use

Use when preparing scRNA-seq count matrices (mESC, melanoma Mel78-88, hESC 10X Chromium) as input to Cyclum's circular-manifold autoencoder or comparators like reCAT, Seurat, or ccRemover.

## When NOT to use

- Input is already a normalized/feature-scaled expression matrix rather than raw counts — re-applying log2 and variance scaling would distort the data
- The downstream method requires its own normalization scheme (e.g., methods outside the Cyclum/reCAT comparison set that specify different transforms)

## Inputs

- Raw scRNA-seq gene-by-cell count matrix (e.g., mESC matrix from cyclum2 notebook data, 10X Chromium count matrices for Mel78-88 or hESC)
- Multiple matrices to be compared (treated vs. control, down-sampled replicates) that must receive identical preprocessing

## Outputs

- Scaled, centered, log2(x+1)-transformed expression matrix ready for Cyclum, reCAT, Seurat, or ccRemover
- Consistently normalized matrices feeding downstream outputs such as pseudotime_2pi.csv, accuracy_comparison.csv, or gene_circular_magnitude_ranking.csv

## How to apply

Apply the transform to each raw expression matrix before fitting any model: compute log2(x+1) per count, then scale each gene to variance = 1 and center to mean = 1, as recommended in the Cyclum FAQ/methods. Apply the identical preprocessing to all matrices that will be compared (e.g., Cyclum vs. reCAT on the same down-sampled mESC data, or treated vs. control hESC 10X matrices) so downstream pseudo-time inference and circular-pattern scores are comparable. The Cyclum autoencoder then projects these preprocessed data to a circular manifold using a tanh-encoder/cos-sin-decoder network trained with least-square loss plus L2 regularization in Keras/TensorFlow. Downstream tasks that depend on this normalization include pseudo-time inference, cell-cycle effect removal, and gene circular-pattern magnitude computation via cyclum.evaluation.

## Related tools

- **Cyclum** (Primary consumer of the preprocessed matrix; its sinusoidal autoencoder requires scaled (variance=1), centered (mean=1) log2(x+1) data before projecting cells onto the circular manifold) — https://github.com/KChen-lab/Cyclum
- **reCAT** (Comparator method run on identically preprocessed matrices (e.g., down-sampled mESC) for fair accuracy comparison)
- **cyclum.tuning.CyclumAutoTune** (Applied after preprocessing to select the optimal number of linear dimensions for the circular manifold) — https://github.com/lshh125/cyclum2
- **ccRemover** (Linear cell-cycle regression baseline run on the same preprocessed matrices in correction comparisons)
- **Seurat** (Linear regression baseline consuming the identically preprocessed matrices for cell-cycle correction comparisons)

## Examples

```
X = np.log2(raw_counts + 1); X = (X - X.mean(axis=0)) / X.std(axis=0); X = (X - X.mean(axis=0)) + 1  # scale to var=1, center to mean=1
```

## Evaluation signals

- Preprocessed matrix has per-gene variance = 1 and mean = 1 after the log2(x+1) transform (verifiable with numpy checks on the scaled data)
- All matrices destined for method comparison (Cyclum vs. reCAT vs. Seurat vs. ccRemover) show identical scaling statistics — no method sees differently normalized inputs
- Downstream sanity: Cyclum run on correctly preprocessed mESC data yields median cell-cycle classification accuracy within the expected 0.70-0.75 band across down-sampling conditions
- On Mel78, the uncorrected-vs-corrected AXL/MITF correlation (R = -0.02, P = 0.81 uncorrected; R = -0.43, P = 9e-7 Cyclum-corrected) is reproducible only when preprocessing matches the paper's transform
- Wrapped pseudo-time values fall within [0, 2π] after modulus, confirming the autoencoder converged on the normalized input

## Limitations

- No version numbers for Python, Keras, or TensorFlow are stated in the article; the README pins tested versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0) that differ from latest pip defaults
- The transform recommendation is heuristic ('work well') from the Cyclum FAQ/methods, not derived from an optimization study — other scalings may also be valid
- TensorFlow < 1.15.2 has a known security issue per the README, constraining environment setup

## Evidence

- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [supplementary] preprocess with a log2(x+1) transform, scaling to variance = 1 and centering (mean = 1), as recommended in the Cyclum FAQ: [evidence span withheld — anchor status: not_found]
- [readme] Cyclum was tested on these package versions. Please make sure that you have TensorFlow 1.x.: [evidence span withheld — anchor status: external_doc]
- [methods] It provides methods to recover cell cycle information and remove cell cycle effects from the scRNA-seq data.: "It provides methods to recover cell cycle information and remove cell cycle effects from the scRNA-seq data."
