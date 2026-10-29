---
name: sugar-diffusion-scrna-generation
description: Reference for comparing deep-learning scRNA-seq generators (scGAN/cscGAN)
  against SUGAR and Splatter on distributional fidelity (MMD) and realism (RF-AUC)
  on multi-cell-type data such as PBMCs.
when_to_use_negative:
- When the goal is conditional generation of a specific cell type or subpopulation
  on demand — that requires cscGAN, not SUGAR-based unconditional manifold generation.
- When the input data is not in .h5/.h5ad (or has not been converted via the provided
  Jupyter notebook) and no pre-processing step has been run.
- When only a single homogeneous cell population is present and the multi-cell-type
  fidelity benchmark (MMD/RF-AUC vs SUGAR/Splatter) is not the analysis goal.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SUGAR
  role: Geometry-based data generator used as the state-of-the-art comparison method;
    generates synthetic cells along the data manifold via diffusion random-walks
  repo: https://github.com/KrishnaswamyLab/SUGAR
- name: scGAN
  role: GAN-based generator producing the synthetic cells being benchmarked against
    SUGAR and Splatter
  repo: https://github.com/imsb-uke/scGAN
- name: Splatter
  role: Baseline scRNA-seq simulation tool run on the same PBMC training cells for
    MMD/RF-AUC comparison
- name: Docker
  role: Provides the reproducible environment (Dockerfile/image) for running the scGAN
    pipeline and comparisons
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Backend framework in which the scGAN models are implemented and run
- name: Python
  role: Runtime for scGAN (tested with Python 3.5) and SUGAR (implemented in Python3)
    pipelines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:47db6f5409cb9ae01f43ec5b6fbb95a6a1292c6757931aae23dfa9efbd4efc7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KrishnaswamyLab/SUGAR
  tools:
  - Docker
  - Python
  - SUGAR
  - Splatter
  - TensorFlow
  - cscGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# sugar-diffusion-scrna-generation

## Summary

Benchmark skill for running SUGAR (Synthesis Using Geometrically Aligned Random-walks) to generate synthetic scRNA-seq cells from a real training set (e.g. Fresh 68k PBMCs Donor A) and evaluating them against scGAN and Splatter using MMD and random-forest AUC (RF-AUC).

## When to use

Reference for comparing deep-learning scRNA-seq generators (scGAN/cscGAN) against SUGAR and Splatter on distributional fidelity (MMD) and realism (RF-AUC) on multi-cell-type data such as PBMCs.

## When NOT to use

- When the goal is conditional generation of a specific cell type or subpopulation on demand — that requires cscGAN, not SUGAR-based unconditional manifold generation.
- When the input data is not in .h5/.h5ad (or has not been converted via the provided Jupyter notebook) and no pre-processing step has been run.
- When only a single homogeneous cell population is present and the multi-cell-type fidelity benchmark (MMD/RF-AUC vs SUGAR/Splatter) is not the analysis goal.

## Inputs

- Pre-processed scRNA-seq training cells in .h5 or .h5ad format (e.g. Fresh 68k PBMCs (Donor A) dataset)
- Trained scGAN model checkpoint (for the comparison arm)
- parameters. parameter file for the scGAN run

## Outputs

- SUGAR-generated synthetic count matrix matched in size to the real training cells
- scGAN-generated cells (e.g. scgan_cells.h5ad) and Splatter-simulated count matrix
- MMD values for each method (Splatter, SUGAR, scGAN) versus the real training cells
- RF-AUC values from random forest real-vs-generated discrimination per method
- mmd_rf_auc_benchmark.csv summarizing MMD and RF-AUC per method

## How to apply

Load the pre-processed Fresh 68k PBMCs (Donor A) training cells in .h5/.h5ad format, and run SUGAR (KrishnaswamyLab/SUGAR), which learns manifold geometry via a diffusion process and generates new points evenly along the manifold by pulling random points into its intrinsic structure with a diffusion kernel, to produce a synthetic dataset matched in size to the training cells. In parallel, generate a matched set of cells with a trained scGAN model (`--generate --cells_no ...`) and simulate with Splatter on the same input cells. Compute the maximum mean discrepancy (MMD) between the real training cells and each method's generated cells, and train a random forest classifier to discriminate real versus generated cells for each method, recording the resulting RF-AUC. Use real-data positive controls (expected MMD ~0.03 and AUC ~0.52, i.e. near-chance discrimination) to calibrate the metrics. Judge correctness by compiling results into mmd_rf_auc_benchmark.csv and checking against Supplementary Tables 2-3: Splatter MMD 129.52; SUGAR MMD 59.45 / AUC 0.98; scGAN MMD 0.87 / AUC 0.65 — lower MMD and AUC near the positive control indicate more realistic generation.

## Related tools

- **SUGAR** (Geometry-based data generator used as the state-of-the-art comparison method; generates synthetic cells along the data manifold via diffusion random-walks) — https://github.com/KrishnaswamyLab/SUGAR
- **scGAN** (GAN-based generator producing the synthetic cells being benchmarked against SUGAR and Splatter) — https://github.com/imsb-uke/scGAN
- **Splatter** (Baseline scRNA-seq simulation tool run on the same PBMC training cells for MMD/RF-AUC comparison)
- **Docker** (Provides the reproducible environment (Dockerfile/image) for running the scGAN pipeline and comparisons) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Backend framework in which the scGAN models are implemented and run)
- **Python** (Runtime for scGAN (tested with Python 3.5) and SUGAR (implemented in Python3) pipelines)

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path scgan_cells.h5ad
```

## Evaluation signals

- MMD between real training cells and SUGAR-generated cells should land near the reference value 59.45 (Supplementary Table 2-3), far above scGAN's 0.87 and the positive control 0.03.
- RF-AUC for discriminating real vs SUGAR-generated cells should be near 0.98; a well-behaved generator (scGAN) yields ~0.65 and real-vs-real positive control ~0.52.
- Real-data positive controls must reproduce MMD ~0.03 and AUC ~0.52; failure here invalidates the metric computation.
- mmd_rf_auc_benchmark.csv must contain matching Splatter/SUGAR/scGAN rows whose values equal the Supplementary Table references (e.g. Splatter MMD 129.52).
- SUGAR output must contain the same number of cells as requested from the training set (equally sized synthetic dataset).

## Limitations

- SUGAR's generated cells are easily discriminable from real data (RF-AUC 0.98, MMD 59.45), indicating poor realism compared to scGAN on this PBMC benchmark.
- Comparability requires all methods to be trained/simulated on the same pre-processed PBMC training cells; differing inputs break the MMD/RF-AUC comparison.
- MMD and RF-AUC depend on the specific pre-processing (filtering parameters, train/validation/test split defined in parameters.); results are only meaningful when the same processed data is used across methods.

## Evidence

- [readme] SUGAR uses a diffusion process to learn a manifold geometry from the data. Then, it generates new points evenly along the manifold: [evidence span withheld — anchor status: external_doc]
- [methods] most of the experiments reported in our paper use the Fresh 68k PBMCs (Donor A) dataset: "most of the experiments reported in our paper use the Fresh 68k PBMCs (Donor A) dataset"
- [discussion] cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells: "cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells."
- [readme] we also provide a Dockerfile that contains a working environment containing all the dependencies: "To allow you to reproduce our results easily, we also provide a Dockerfile that contains a working environment containing all the dependencies."
- [readme] Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format: "Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format. In case you want to use data in mtx/tsv, please use the provided Jupyter notebook"
