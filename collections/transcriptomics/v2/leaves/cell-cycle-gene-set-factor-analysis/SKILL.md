---
name: cell-cycle-gene-set-factor-analysis
description: Use when labeled (Hoechst or equivalent) scRNA-seq or qPCR single-cell
  data are available and the goal is to quantify how accurately different cell-cycle
  characterization methods recover cell-cycle phase labels.
when_to_use_negative:
- The data lack any ground-truth cell-cycle labels (e.g. Hoechst staining), so supervised
  classification accuracy cannot be computed — use pseudo-time inference alone instead.
- The goal is removing cell-cycle confounding effects from expression data (Cyclum's
  deconvolution mode) rather than benchmarking characterization accuracy.
- The input is already a low-dimensional factor/pseudo-time table; classification
  against labels can be done directly without re-fitting Cyclum.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects preprocessed expression data to a circular
    manifold, infers pseudo-time, and yields cell-cycle classifications to benchmark
    against baselines
  repo: https://github.com/KChen-lab/Cyclum
- name: f-scLVM (Slalom)
  role: Gene-set-based linear latent-factor baseline run with known gene sets to produce
    phase factors for accuracy comparison
- name: Cyclone
  role: Baseline using PCA and relative expression of gene pairs to predict cell-cycle
    phases from the same preprocessed data
- name: reCAT
  role: Model-free baseline reconstructing cell-cycle pseudo-time with a Gaussian
    mixture model
- name: Oscope
  role: Baseline checking pairs of genes to identify circular expression patterns
- name: Seurat
  role: Linear-representation cell-cycle regression approach cited as contrast to
    Cyclum's nonlinear circular manifold
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:af360c24e09797497d82392a11a98ffe825782179d9e3e5f4bafd08b53a468cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclone
  - Cyclum
  - Oscope
  - Seurat
  - f-scLVM (Slalom)
  - reCAT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-cycle-gene-set-factor-analysis

## Summary

Benchmark cell-cycle stage characterization from scRNA-seq/qPCR data by fitting Cyclum's sinusoidal autoencoder and gene-set/linear factor baselines (Slalom/f-scLVM, Cyclone, reCAT, Oscope, Cyclops, PCA) to Hoechst-labeled datasets, then comparing classification accuracy per method per dataset.

## When to use

Use when labeled (Hoechst or equivalent) scRNA-seq or qPCR single-cell data are available and the goal is to quantify how accurately different cell-cycle characterization methods recover cell-cycle phase labels.

## When NOT to use

- The data lack any ground-truth cell-cycle labels (e.g. Hoechst staining), so supervised classification accuracy cannot be computed — use pseudo-time inference alone instead.
- The goal is removing cell-cycle confounding effects from expression data (Cyclum's deconvolution mode) rather than benchmarking characterization accuracy.
- The input is already a low-dimensional factor/pseudo-time table; classification against labels can be done directly without re-fitting Cyclum.

## Inputs

- Hoechst-labeled scRNA-seq or single-cell qPCR expression matrices for mESC, PC3, MDA-MB-231, and H9 datasets
- Known cell-cycle gene sets for gene-set-based baselines (f-scLVM/Slalom, Cyclone)

## Outputs

- classification_accuracy_comparison.csv: accuracy table with rows = methods (Cyclum, Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA, NMF, ICA) and columns = the four labeled datasets
- Per-cell inferred pseudo-time / phase assignments from each method

## How to apply

Preprocess each expression matrix with the Cyclum-recommended transform: log2(x+1), then center to mean 1 and scale to variance 1. Fit the Cyclum sinusoidal autoencoder to project cells onto the circular manifold and infer per-cell pseudo-time, using cyclum.tuning.CyclumAutoTune to select the number of linear dimensions. Run baselines on the same preprocessed data — f-scLVM (Slalom) with known gene sets, Cyclone (PCA plus relative expression of gene pairs), reCAT (Gaussian-mixture-model pseudo-time), Oscope (pairwise circular gene patterns), Cyclops, and linear decompositions PCA, NMF, and ICA. Classify cells into phases using each method's pseudo-time or factor scores against the Hoechst-based labels and compute classification accuracy per method per dataset. Assemble accuracies into a methods-by-datasets table (CSV) and verify Cyclum's accuracy exceeds every baseline's on mESC, PC3, MDA-MB-231 and H9.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects preprocessed expression data to a circular manifold, infers pseudo-time, and yields cell-cycle classifications to benchmark against baselines) — https://github.com/KChen-lab/Cyclum
- **f-scLVM (Slalom)** (Gene-set-based linear latent-factor baseline run with known gene sets to produce phase factors for accuracy comparison)
- **Cyclone** (Baseline using PCA and relative expression of gene pairs to predict cell-cycle phases from the same preprocessed data)
- **reCAT** (Model-free baseline reconstructing cell-cycle pseudo-time with a Gaussian mixture model)
- **Oscope** (Baseline checking pairs of genes to identify circular expression patterns)
- **Seurat** (Linear-representation cell-cycle regression approach cited as contrast to Cyclum's nonlinear circular manifold)

## Examples

```
conda create -n cyclum python=3.7 pip && conda activate cyclum && git clone https://github.com/KChen-lab/Cyclum.git && cd Cyclum && pip install .
```

## Evaluation signals

- classification_accuracy_comparison.csv contains accuracy values in [0, 1] for all nine methods across all four datasets (mESC, PC3, MDA-MB-231, H9).
- Cyclum's accuracy exceeds every baseline's accuracy on each of the four datasets, consistent with the reported finding.
- Cyclum specifically outperforms the model-free approach reCAT for cell-cycle characterization.
- Preprocessing invariants: expression values are log2(x+1)-transformed, centered (mean = 1) and scaled (variance = 1) before fitting any model.
- The number of Cyclum linear dimensions was selected via CyclumAutoTune rather than fixed arbitrarily.

## Limitations

- Cyclum requires TensorFlow 1.x and was tested only with specific dependency versions (e.g. python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5); pip installing latest versions may break the setup.
- No version numbers for Python/Keras/TensorFlow are stated in the article's discussion, and hardware/runtime details are only informally reported (Supplementary Note 2).
- Cyclum is model-based (fits data to predefined circular manifolds), so it assumes the periodic process is well represented by a circular trajectory; baselines leveraging known cell-cycle genes (Cyclone, reCAT) use prior information Cyclum does not.

## Evidence

- [abstract] Cyclum substantially improves the accuracy and robustness of cell-cycle characterization beyond existing approaches: "Cyclum substantially improves the accuracy and robustness of cell-cycle characterization beyond existing approaches"
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed data work well: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [intro] Cyclone24 uses PCA and relative expression of gene pairs to predict cell-cycle phases: "Cyclone24 uses PCA and relative expression of gene pairs to predict cell-cycle phases"
- [discussion] It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles.: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [readme] we also implemented class cyclum.tuning.CyclumAutoTune, which automatically select the proper number of linear components: [evidence span withheld — anchor status: external_doc]
