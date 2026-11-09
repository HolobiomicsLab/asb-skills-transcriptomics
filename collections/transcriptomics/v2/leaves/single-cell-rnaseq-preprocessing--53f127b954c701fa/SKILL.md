---
name: single-cell-rnaseq-preprocessing
description: Use when droplet-based 10X Chromium scRNA-seq data (mouse aorta light-dark
  cycle, hepatocyte time course) must be loaded as anndata objects and subset to cell
  types for circadian phase inference.
when_to_use_negative:
- Input is bulk RNA-seq or plate-based Fluidigm C1 data whose likelihood assumptions
  differ from the droplet-based generative model this preprocessing targets
- The goal is circadian phase inference itself rather than preparing the anndata inputs
  consumed by the inference step
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: Preprocessing and inference environment whose conda dependencies (Python >=
    3.8, scanpy>=1.6, pytorch>=1.9.0, anndata) define the preprocessing stack; validated
    via run_test.py
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: Container format for loading the aorta and hepatocyte datasets and subsetting
    aorta cells to SMCs, fibroblasts, endothelial cells, and macrophages
- name: scanpy
  role: Single-cell reading and manipulation library (scanpy>=1.6) required by the
    tempo environment for handling the anndata objects
- name: 10X Genomics Chromium
  role: Source of the droplet-based mouse aorta light-dark scRNA-seq data that this
    preprocessing step loads and subsets
- name: pytorch
  role: Backend (pytorch>=1.9.0) for Tempo's Bayesian variational inference after
    preprocessing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:eb035a256ba13192e75dd6d2cdd2e44b87a2a754a0424fbfdd3ab36e0a69f1af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10X Genomics Chromium
  - Cyclops
  - Cyclum
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# single-cell-rnaseq-preprocessing

## Summary

Prepare droplet-based single-cell RNA-seq data (e.g., 10X Chromium mouse aorta light-dark and Droin et al. hepatocyte time-course datasets) as anndata objects inside the Tempo conda environment so that circadian phase-inference methods can be benchmarked against known sampling-time references.

## When to use

Use when droplet-based 10X Chromium scRNA-seq data (mouse aorta light-dark cycle, hepatocyte time course) must be loaded as anndata objects and subset to cell types for circadian phase inference.

## When NOT to use

- Input is bulk RNA-seq or plate-based Fluidigm C1 data whose likelihood assumptions differ from the droplet-based generative model this preprocessing targets
- The goal is circadian phase inference itself rather than preparing the anndata inputs consumed by the inference step

## Inputs

- 10X Chromium mouse aorta light-dark scRNA-seq dataset (droplet-based)
- Droin et al. mouse hepatocyte time-course scRNA-seq dataset
- anndata object with cell-type annotations for aorta SMCs, fibroblasts, endothelial cells, and macrophages

## Outputs

- anndata objects per cell type and per dataset, ready for Tempo / Cyclops / Cyclum / PCA input
- per-cell phase estimates in hours for downstream median-error and Bayes-factor computation
- phase_benchmark_results.csv aggregating median phase errors and log Bayes factors

## How to apply

Build the tempo conda environment (Python >= 3.8 with scanpy>=1.6, pytorch>=1.9.0, anndata, numpy, pandas, scikit-learn, scipy) by cloning github.com/bauerbach95/tempo and running source install.sh, then activate it with conda activate tempo. Validate the install by running python run_test.py and confirming the 'SUCCESSFULLY FINISHED' message before any real analysis. Load the 10X Chromium mouse aorta light-dark scRNA-seq dataset and the Droin et al. hepatocyte time-course dataset as anndata objects, subsetting the aorta data to SMCs, fibroblasts, endothelial cells, and macrophages so each cell type can be analyzed independently. The rationale for droplet-aware handling is that existing unsupervised phase-inference methods were developed for plate-based data (Fluidigm C1) whose likelihood models approximate droplet-based generative distributions poorly, so preprocessing choices must preserve the raw count structure that Tempo's Bayesian variational model expects. Preprocessed objects feed downstream per-cell phase estimation, where Tempo's accuracy is judged by median phase error against known sampling times (e.g., 1.88 h for aorta SMCs).

## Related tools

- **Tempo** (Preprocessing and inference environment whose conda dependencies (Python >= 3.8, scanpy>=1.6, pytorch>=1.9.0, anndata) define the preprocessing stack; validated via run_test.py) — github.com/bauerbach95/tempo
- **anndata** (Container format for loading the aorta and hepatocyte datasets and subsetting aorta cells to SMCs, fibroblasts, endothelial cells, and macrophages)
- **scanpy** (Single-cell reading and manipulation library (scanpy>=1.6) required by the tempo environment for handling the anndata objects)
- **10X Genomics Chromium** (Source of the droplet-based mouse aorta light-dark scRNA-seq data that this preprocessing step loads and subsets)
- **pytorch** (Backend (pytorch>=1.9.0) for Tempo's Bayesian variational inference after preprocessing)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- python run_test.py completes in under 5 minutes and prints 'SUCCESSFULLY FINISHED'
- Conda environment contains Python >= 3.8 with the documented dependencies: anndata, numpy, pandas, scanpy>=1.6, scikit-learn, scipy, pytorch>=1.9.0
- Loaded aorta anndata object subsets cleanly into SMCs, fibroblasts, endothelial cells, and macrophages; hepatocyte time course retains sampling-time metadata for reference phases
- Downstream Tempo run on the preprocessed all-genes SMC subset reproduces the Table 1 median phase error of 1.88 h
- Resulting phase_benchmark_results.csv shows Tempo log Bayes factors of the reported orders of magnitude (10^17896 SMCs, 10^2257 fibroblasts, 10^110 endothelial cells, 10^184 macrophages, 10^1761 hepatocytes)

## Limitations

- Tempo installation requires mini-conda and has only been tested on macOS 10.14.5 (Mojave), macOS 12.2.1 (Monterey), and CentOS Linux 7; >=8GB RAM is recommended for in-memory operations
- No software version for Tempo is stated, so environment reproducibility depends on pinning the repository state at clone time
- Existing unsupervised phase-inference methods run poorly on circadian data because the core clock comprises only ~20 moderately expressed genes; preprocessing choices must not discard these genes

## Evidence

- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [intro] droplet vs plate-based capture: "Relative to droplet-based techniques (e.g., 10X Genomics Chromium), plate-based approaches tend to capture fewer cells"
- [readme] environment activation: "After installing, you can activate the conda environment containing the installed Tempo package: conda activate tempo"
- [readme] smoke-test validation: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [intro] core clock gene problem: "existing unsupervised phase inference methods perform poorly when tasked with ordering cells over the circadian cycle."
