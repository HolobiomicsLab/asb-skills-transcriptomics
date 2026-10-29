---
name: phase-classification-accuracy-evaluation
description: Use when scRNA-seq or qPCR single-cell data have Hoechst-based (or similar
  experimental) cell-cycle phase labels and you need to score and compare cell-cycle
  classification accuracy of competing pseudo-time methods.
when_to_use_negative:
- Data lack experimental phase labels (e.g., Hoechst staining), so classification
  accuracy against ground truth cannot be computed
- The goal is deconvolving/regressing out cell-cycle effects to improve subpopulation
  delineation rather than benchmarking classification accuracy
- The biological process is non-periodic (e.g., linear differentiation trajectories
  better handled by Monocle or Wanderlust)
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects expression data onto a circular manifold
    to infer per-cell pseudo-time for phase classification; CyclumAutoTune selects
    the number of linear dimensions
  repo: https://github.com/KChen-lab/Cyclum
- name: f-scLVM (Slalom)
  role: Baseline using known gene sets to infer latent factors for phase classification
- name: Cyclone
  role: Baseline using PCA plus relative expression of gene pairs to predict cell-cycle
    phases
- name: reCAT
  role: Baseline reconstructing cell-cycle pseudo-time with a Gaussian mixture model
    (model-free comparison)
- name: Oscope
  role: Baseline checking pairs of genes to identify circular patterns
- name: PCA
  role: Linear decomposition baseline (PC1/PC2 scores used for phase classification)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:e5bf182602ca5e4a83205cb74d4c25573a0c97afa1695874701751890214fb5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclone
  - Cyclum
  - Oscope
  - PCA
  - f-scLVM (Slalom)
  - reCAT
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# phase-classification-accuracy-evaluation

## Summary

Benchmark a cell-cycle inference method by classifying cells into Hoechst-derived cell-cycle phases and comparing classification accuracy against baselines (Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA/NMF/ICA). It quantifies whether model-based circular manifold inference (Cyclum) outperforms linear and model-free approaches.

## When to use

Use when scRNA-seq or qPCR single-cell data have Hoechst-based (or similar experimental) cell-cycle phase labels and you need to score and compare cell-cycle classification accuracy of competing pseudo-time methods.

## When NOT to use

- Data lack experimental phase labels (e.g., Hoechst staining), so classification accuracy against ground truth cannot be computed
- The goal is deconvolving/regressing out cell-cycle effects to improve subpopulation delineation rather than benchmarking classification accuracy
- The biological process is non-periodic (e.g., linear differentiation trajectories better handled by Monocle or Wanderlust)

## Inputs

- Hoechst-labeled scRNA-seq/qPCR expression matrices from mESC, PC3, MDA-MB-231 and H9 datasets
- Per-cell Hoechst-based phase labels (G1/S/G2-M) as ground truth
- Known cell-cycle gene sets (for f-scLVM/Slalom and Cyclone baselines)

## Outputs

- classification_accuracy_comparison.csv: a table of classification accuracy (rows = nine methods, columns = four datasets)
- Per-cell phase assignments (G1/S/G2-M) from each method's pseudo-time or factor scores

## How to apply

Preprocess each labeled dataset (mESC, PC3, MDA-MB-231, H9) with the recommended transform: log2(x+1) followed by centering (mean = 1) and scaling (variance = 1). Fit the Cyclum sinusoidal autoencoder to project cells onto the circular manifold and infer per-cell pseudo-time, using cyclum.tuning.CyclumAutoTune to select the number of linear dimensions. Run baselines on the same preprocessed data — f-scLVM (Slalom) with known gene sets, Cyclone (PCA plus relative expression of gene pairs), reCAT (GMM pseudo-time), Oscope (pairwise circular gene patterns), Cyclops, and linear decompositions PCA, NMF and ICA. Classify cells into phases using each method's pseudo-time or factor scores against the Hoechst labels, compute accuracy per method per dataset, and assemble a methods-by-datasets table. Rationale: Cyclum's model-based fit to predefined circular manifolds is expected to be more robust than model-free approaches such as reCAT, so the key decision point is confirming Cyclum's accuracy exceeds every baseline on all four datasets.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects expression data onto a circular manifold to infer per-cell pseudo-time for phase classification; CyclumAutoTune selects the number of linear dimensions) — https://github.com/KChen-lab/Cyclum
- **f-scLVM (Slalom)** (Baseline using known gene sets to infer latent factors for phase classification)
- **Cyclone** (Baseline using PCA plus relative expression of gene pairs to predict cell-cycle phases)
- **reCAT** (Baseline reconstructing cell-cycle pseudo-time with a Gaussian mixture model (model-free comparison))
- **Oscope** (Baseline checking pairs of genes to identify circular patterns)
- **PCA** (Linear decomposition baseline (PC1/PC2 scores used for phase classification))

## Examples

```
tuner = cyclum.tuning.CyclumAutoTune(mtx); n_linear = tuner.prompt_num_of_components(); model = cyclum.models.AutoEncoder(mtx, n_linear); pseudotime = model.train().calculate_pseudotime()
```

## Evaluation signals

- classification_accuracy_comparison.csv contains accuracy values in [0, 1] for all nine methods across all four datasets (mESC, PC3, MDA-MB-231, H9)
- Cyclum's accuracy strictly exceeds every baseline (Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA, NMF, ICA) on each of the four datasets
- Preprocessing invariants: each input matrix is log2(x+1)-transformed, centered to mean = 1 and scaled to variance = 1 before fitting
- CyclumAutoTune reports a selected number of linear dimensions per dataset and the fitted manifold is validated as 'most circular'
- Per-cell pseudo-time yields exactly one inferred stage per cell, and phase assignments cover all Hoechst-labeled cells

## Limitations

- Cyclum assumes the latent process is circular; linear trajectory methods (Monocle, Wanderlust) target non-periodic processes and are not appropriate comparisons for periodic accuracy
- No software version numbers for Python/Keras/TensorFlow were stated in the paper; README reports tested versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0) and notes TensorFlow < 1.15.2 has a known security issue
- The comparison depends on baseline implementations leveraging known cell-cycle genes (Cyclone, reCAT), which may differ from original publications' settings

## Evidence

- [methods] Preprocessing transform: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum vs reCAT advantage: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [discussion] Robustness rationale: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [intro] Baseline method descriptions: "Cyclone24 uses PCA and relative expression of gene pairs to predict cell-cycle phases"
- [readme] AutoTune usage: [evidence span withheld — anchor status: external_doc]
