---
name: baseline-method-benchmarking-across-datasets
description: Use when a new latent-process or pseudo-time method must be shown to
  outperform existing methods across several labeled scRNA-seq datasets with a shared
  accuracy metric and comparison table.
when_to_use_negative:
- Datasets lack Hoechst (or equivalent) ground-truth phase labels, so classification
  accuracy against truth cannot be computed
- The analysis goal is pseudo-time or gene-set discovery itself rather than comparing
  methods head-to-head; use the primary method skill instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Proposed method under evaluation; sinusoidal autoencoder that projects cells
    onto a circular manifold and infers per-cell pseudo-time
  repo: https://github.com/KChen-lab/Cyclum
- name: CyclumAutoTune
  role: cyclum.tuning.CyclumAutoTune selects the number of linear dimensions of the
    circular manifold before fitting
  repo: https://github.com/KChen-lab/Cyclum
- name: f-scLVM (Slalom)
  role: Baseline using known gene sets for latent factor inference
- name: Cyclone
  role: Baseline using PCA plus relative expression of gene pairs to predict cell-cycle
    phases
- name: reCAT
  role: Baseline reconstructing cell-cycle pseudo-time with a Gaussian mixture model
- name: Oscope
  role: Baseline checking pairs of genes to identify circular expression patterns
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:3f903d9e7ac3aa57a5fdb700e09c4f798237751546f6f6addf9efce68bde7cee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclone
  - Cyclum
  - CyclumAutoTune
  - Oscope
  - f-scLVM (Slalom)
  - reCAT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# baseline-method-benchmarking-across-datasets

## Summary

A benchmarking skill for comparing a proposed single-cell method (e.g., Cyclum) against baselines (Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA/NMF/ICA) on multiple labeled datasets, measuring cell-cycle classification accuracy against Hoechst-based ground truth.

## When to use

Use when a new latent-process or pseudo-time method must be shown to outperform existing methods across several labeled scRNA-seq datasets with a shared accuracy metric and comparison table.

## When NOT to use

- Datasets lack Hoechst (or equivalent) ground-truth phase labels, so classification accuracy against truth cannot be computed
- The analysis goal is pseudo-time or gene-set discovery itself rather than comparing methods head-to-head; use the primary method skill instead

## Inputs

- Four Hoechst-labeled scRNA-seq/qPCR cell-cycle datasets: mESC, PC3, MDA-MB-231, H9 (expression matrices plus Hoechst phase labels)
- Preprocessed matrices: log2(x+1)-transformed, centered (mean = 1), scaled (variance = 1)

## Outputs

- classification_accuracy_comparison.csv (rows = methods, columns = datasets, values = classification accuracy in [0, 1])
- Per-dataset pseudo-time/phase assignments for each of the nine methods (Cyclum, Slalom/f-scLVM, Cyclone, reCAT, Oscope, Cyclops, PCA, NMF, ICA)

## How to apply

Preprocess every dataset identically with the log2(x+1) transform followed by centering (mean = 1) and scaling (variance = 1), as recommended in the Cyclum documentation. Fit the proposed method (Cyclum sinusoidal autoencoder, tuning the number of linear dimensions with cyclum.tuning.CyclumAutoTune) and run all baselines on the same preprocessed matrices: f-scLVM/Slalom with known gene sets, Cyclone (PCA plus relative expression of gene pairs), reCAT (Gaussian-mixture-model pseudo-time), Oscope (pairwise circular gene patterns), Cyclops, and linear decompositions PCA (PC1/PC2), NMF, and ICA. Classify cells into phases using each method's pseudo-time/factor scores against the Hoechst-derived phase labels and compute classification accuracy per method per dataset. Assemble a methods-by-datasets accuracy table (e.g., classification_accuracy_comparison.csv) covering the mESC, PC3, MDA-MB-231, and H9 Hoechst-labeled datasets, then check the headline claim (proposed method exceeds every baseline on all datasets).

## Related tools

- **Cyclum** (Proposed method under evaluation; sinusoidal autoencoder that projects cells onto a circular manifold and infers per-cell pseudo-time) — https://github.com/KChen-lab/Cyclum
- **CyclumAutoTune** (cyclum.tuning.CyclumAutoTune selects the number of linear dimensions of the circular manifold before fitting) — https://github.com/KChen-lab/Cyclum
- **f-scLVM (Slalom)** (Baseline using known gene sets for latent factor inference)
- **Cyclone** (Baseline using PCA plus relative expression of gene pairs to predict cell-cycle phases)
- **reCAT** (Baseline reconstructing cell-cycle pseudo-time with a Gaussian mixture model)
- **Oscope** (Baseline checking pairs of genes to identify circular expression patterns)

## Examples

```
conda create -n cyclum python=3.7 pip; conda activate cyclum; git clone https://github.com/KChen-lab/Cyclum.git && cd Cyclum && pip install .
```

## Evaluation signals

- All accuracy entries in the table fall within [0, 1] for all nine methods across mESC, PC3, MDA-MB-231, and H9
- Cyclum's accuracy exceeds every baseline's accuracy on each of the four datasets
- Cyclum specifically outperforms the model-free reCAT, consistent with the paper's discussion of model-based circular manifolds being more robust to noise
- Identical preprocessing (log2(x+1), centered mean = 1, scaled variance = 1) was applied to every dataset before running any method
- The comparison table has complete coverage: no missing accuracy values for any method-dataset pair

## Limitations

- Several baselines (Cyclone, reCAT) leverage known cell-cycle genes, while Cyclum is ab initio and marker-gene-free, so accuracy comparisons mix model-based and marker-informed regimes
- Benchmarking was restricted to four Hoechst-labeled datasets; results may not generalize to unlabeled or non-Hoechst-stained systems
- Cyclum was tested on specific dependency versions (TensorFlow 1.14.0, keras 2.2.4, python 3.7.4); newer versions may require forcing pinned installs via setup.py

## Evidence

- [methods] preprocess recommendation: [evidence span withheld — anchor status: not_found]
- [abstract] Cyclum autoencoder rationale: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [discussion] Cyclum vs reCAT finding: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [intro] baseline descriptions: "Cyclone24 uses PCA and relative expression of gene pairs to predict cell-cycle phases"
- [readme] CyclumAutoTune README: [evidence span withheld — anchor status: external_doc]
