---
name: scrna-seq-generative-model-benchmarking
description: 'Decision support for choosing a synthetic scRNA-seq generator: use when
  multiple simulation tools can be trained on the same .h5/.h5ad single-cell count
  matrix and you need quantitative fidelity (MMD) and realism (RF-AUC) rankings.'
when_to_use_negative:
- The goal is generating specific cell types or subpopulations on demand (conditioning)
  rather than comparing generator quality — use the cscGAN conditional generation
  workflow instead
- Input data is not scRNA-seq counts in .h5/.h5ad (e.g. raw mtx/tsv that has not yet
  been converted with the provided Jupyter notebook)
- Each candidate generator would be trained or fitted on different source cells, since
  MMD and RF-AUC comparisons are only valid on the same matched PBMC training cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAN
  role: TensorFlow implementation used to generate the GAN-based synthetic cells being
    benchmarked (--generate flag writes scgan_cells.h5ad)
  repo: https://github.com/imsb-uke/scGAN
- name: Splatter
  role: Baseline scRNA-seq simulation tool run on the same PBMC training cells to
    produce a comparison count matrix (reference MMD 129.52)
- name: SUGAR
  role: Geometry-based (diffusion random-walk) data generation tool run on the same
    training cells; baseline with reference MMD 59.45 and AUC 0.98
  repo: https://github.com/KrishnaswamyLab/SUGAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:8c3c8521a248c655fc32595234d168c02d4d47c58991a3ac8b39fe7fcb8976cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - SUGAR
  - Splatter
  - cscGAN
  - scGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-generative-model-benchmarking

## Summary

A benchmarking protocol for comparing scRNA-seq generative models (scGAN, Splatter, SUGAR) trained on the same cells, using maximum mean discrepancy (MMD) for distributional fidelity and a random-forest discriminator AUC (RF-AUC) for realism.

## When to use

Decision support for choosing a synthetic scRNA-seq generator: use when multiple simulation tools can be trained on the same .h5/.h5ad single-cell count matrix and you need quantitative fidelity (MMD) and realism (RF-AUC) rankings.

## When NOT to use

- The goal is generating specific cell types or subpopulations on demand (conditioning) rather than comparing generator quality — use the cscGAN conditional generation workflow instead
- Input data is not scRNA-seq counts in .h5/.h5ad (e.g. raw mtx/tsv that has not yet been converted with the provided Jupyter notebook)
- Each candidate generator would be trained or fitted on different source cells, since MMD and RF-AUC comparisons are only valid on the same matched PBMC training cells

## Inputs

- Pre-processed Fresh 68k PBMCs (Donor A) training cells in .h5 or .h5ad format
- A trained scGAN model checkpoint (path to model directory)
- Same PBMC training cells passed to Splatter for matched simulation
- Same PBMC training cells passed to SUGAR for diffusion-kernel-based generation

## Outputs

- Synthetic single-cell count matrices per method (e.g. scgan_cells.h5ad from scGAN, Splatter matrix, SUGAR dataset)
- MMD values between real training cells and each method's generated cells
- RF-AUC values from random forest real-vs-generated discrimination per method
- mmd_rf_auc_benchmark.csv compiling MMD and RF-AUC per method

## How to apply

Load the pre-processed Fresh 68k PBMCs (Donor A) training cells in .h5/.h5ad and, if needed, convert mtx/tsv inputs to .h5/.h5ad with the provided Jupyter notebook. Generate a matched synthetic set with the trained scGAN via main.py --generate, then run Splatter and SUGAR (KrishnaswamyLab/SUGAR) on the identical training cells to produce equally sized synthetic count matrices. For each generator, compute the MMD between real training cells and generated cells, and train a random forest to discriminate real versus generated cells, recording the RF-AUC. Compile results into mmd_rf_auc_benchmark.csv and validate against the reference values: scGAN MMD 0.87 / AUC 0.65, SUGAR MMD 59.45 / AUC 0.98, Splatter MMD 129.52, with real-data positive controls at MMD 0.03 / AUC 0.52. The rationale is that low MMD (close to the real-vs-real control) indicates distributional fidelity while RF-AUC near 0.5 indicates generated cells that are hard to distinguish from real ones.

## Related tools

- **scGAN** (TensorFlow implementation used to generate the GAN-based synthetic cells being benchmarked (--generate flag writes scgan_cells.h5ad)) — https://github.com/imsb-uke/scGAN
- **Splatter** (Baseline scRNA-seq simulation tool run on the same PBMC training cells to produce a comparison count matrix (reference MMD 129.52))
- **SUGAR** (Geometry-based (diffusion random-walk) data generation tool run on the same training cells; baseline with reference MMD 59.45 and AUC 0.98) — https://github.com/KrishnaswamyLab/SUGAR

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path scgan_cells.h5ad
```

## Evaluation signals

- scGAN MMD of 0.87 and RF-AUC of 0.65 on Fresh 68k PBMCs (Donor A) should be reproduced when the same training cells and pipeline are used
- SUGAR baseline should yield MMD 59.45 and RF-AUC 0.98; Splatter baseline should yield MMD 129.52
- Real-data positive control (real vs. real split) must give MMD 0.03 and RF-AUC 0.52, bounding the achievable scores
- Each method's synthetic dataset must contain the same number of cells (e.g. --cells_no 1000 500 0 200 per cluster index) so metrics are comparable
- Compiled mmd_rf_auc_benchmark.csv must match the values in Supplementary Tables 2-3

## Limitations

- SUGAR's poor MMD/AUC relative to scGAN means baseline failures should be interpreted as method limitations of the manifold geometry approach on this PBMC dataset, not pipeline errors alone
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established, so benchmark scores for rare-population generation carry uncertainty
- Metrics are computed on a specific dataset (Fresh 68k PBMCs Donor A); rankings may not transfer to other tissues, preprocessing resolutions, or clustering parameters defined in parameters.

## Evidence

- [abstract] cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality: "cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality"
- [readme] Generation with scGAN: "To generate cells and write them to a file, you can use the `--generate` flag."
- [readme] Benchmark dataset: "most of the experiments reported in our paper use the [Fresh 68k PBMCs (Donor A) dataset]"
- [methods] Input format conversion: "please use the provided Jupyter notebook to convert the data to .h5 or .h5ad"
