---
name: splatter-scrna-seq-simulation
description: Use when benchmarking a scRNA-seq data generation method (e.g. scGAN)
  against Splatter on the same real training cells, evaluating with MMD and random-forest
  discrimination AUC (RF-AUC).
when_to_use_negative:
- The goal is conditional, type-aware generation of specific subpopulations — use
  cscGAN rather than Splatter, which is unconditioned in this comparison
- Input data is not scRNA-seq count data in .h5/.h5ad form and has not been pre-processed/split
  alongside the scGAN training set, so a matched comparison is impossible
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Baseline scRNA-seq simulation tool run on the same PBMC training cells to
    produce a synthetic count matrix for MMD/RF-AUC comparison
- name: scGAN
  role: GAN whose generated cells are the method under evaluation in the benchmark
  repo: https://github.com/imsb-uke/scGAN
- name: SUGAR
  role: Geometry-based data generation baseline compared alongside Splatter and scGAN
  repo: https://github.com/KrishnaswamyLab/SUGAR
- name: Python
  role: Environment (tested with Python 3.5) used to run pre-processing, generation,
    and the MMD/RF-AUC evaluation
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Provides the reproducible working environment containing all dependencies
    via the provided Dockerfile
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:06cc5528b3662d80ca586fa8d5073b77b2754d3942b6e3d74f984b7aed87539e
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
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# splatter-scrna-seq-simulation

## Summary

Simulates synthetic scRNA-seq count matrices with Splatter from the same training cells (e.g. Fresh 68k PBMCs Donor A) used to train scGAN, providing a state-of-the-art non-deep-learning baseline for judging the distributional fidelity (MMD) and realism (RF-AUC) of GAN-generated cells.

## When to use

Use when benchmarking a scRNA-seq data generation method (e.g. scGAN) against Splatter on the same real training cells, evaluating with MMD and random-forest discrimination AUC (RF-AUC).

## When NOT to use

- The goal is conditional, type-aware generation of specific subpopulations — use cscGAN rather than Splatter, which is unconditioned in this comparison
- Input data is not scRNA-seq count data in .h5/.h5ad form and has not been pre-processed/split alongside the scGAN training set, so a matched comparison is impossible

## Inputs

- Pre-processed scRNA-seq training cells (.h5 or .h5ad) used to train scGAN (e.g. Fresh 68k PBMCs (Donor A))
- Generated/simulated cell matrices from each method (scGAN via --generate, SUGAR, Splatter)

## Outputs

- Splatter-simulated synthetic scRNA-seq count matrix matched to the real training data
- Benchmark table mmd_rf_auc_benchmark.csv with MMD and RF-AUC for Splatter, SUGAR, and scGAN (reference: Splatter MMD 129.52; SUGAR MMD 59.45 / AUC 0.98; scGAN MMD 0.87 / AUC 0.65)

## How to apply

Run Splatter on the identical pre-processed PBMC training cells (the Fresh 68k PBMCs (Donor A) dataset in .h5/.h5ad format used to train scGAN) to simulate a synthetic count matrix matched in size and gene space to the real data, mirroring how SUGAR (KrishnaswamyLab/SUGAR) and the trained scGAN generate matched synthetic sets. Compute the maximum mean discrepancy (MMD) between the real training cells and Splatter's simulated cells, and train a random forest classifier to discriminate real versus simulated cells, recording the AUC (RF-AUC). Compile all methods' metrics (Splatter, SUGAR, scGAN) into a mmd_rf_auc_benchmark.csv and check against Supplementary Tables 2-3 reference values: Splatter MMD 129.52, SUGAR MMD 59.45 / AUC 0.98, scGAN MMD 0.87 / AUC 0.65, with real-data positive controls at MMD 0.03 and AUC 0.52. A lower MMD and an RF-AUC close to 0.5 indicate the simulator produces realistic cells.

## Related tools

- **Splatter** (Baseline scRNA-seq simulation tool run on the same PBMC training cells to produce a synthetic count matrix for MMD/RF-AUC comparison)
- **scGAN** (GAN whose generated cells are the method under evaluation in the benchmark) — https://github.com/imsb-uke/scGAN
- **SUGAR** (Geometry-based data generation baseline compared alongside Splatter and scGAN) — https://github.com/KrishnaswamyLab/SUGAR
- **Python** (Environment (tested with Python 3.5) used to run pre-processing, generation, and the MMD/RF-AUC evaluation) — https://github.com/imsb-uke/scGAN
- **Docker** (Provides the reproducible working environment containing all dependencies via the provided Dockerfile) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- Splatter MMD against the real PBMC training cells should be near the Supplementary Table 2 reference value of 129.52 (much worse than scGAN's 0.87 and the real-vs-real control of 0.03)
- RF-AUC for Splatter-simulated cells should be computed alongside the references (SUGAR AUC 0.98, scGAN AUC 0.65, real-data positive control AUC 0.52) and compiled into mmd_rf_auc_benchmark.csv
- All compared datasets (real training cells, scGAN-generated, SUGAR, Splatter) must derive from the same pre-processed Fresh 68k PBMCs (Donor A) cells and be of matched size
- The scGAN-generated comparison set must be produced with the documented --generate flag (e.g. --cells_no 1000 500 0 200) from a saved trained model checkpoint

## Limitations

- The scGAN model can generate only per-cluster cell counts specified via --cells_no (e.g. passing 0 for a cluster index generates none), so matched-size simulation requires care
- The test set is not used by the scGAN pipeline ('The test set is not used.'); the comparison rests on the training cells and validation-set t-SNE plots
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established, so simulator fidelity at very small population sizes is uncertain

## Evidence

- [readme] most of the experiments reported in our paper use the Fresh 68k PBMCs (Donor A) dataset: "most of the experiments reported in our paper use the [Fresh 68k PBMCs (Donor A) dataset](http://cf.10xgenomics.com/samples/cell-exp/1.1.0/fresh_68k_pbmc_donor_a/fresh_68k_pbmc_donor_a_filtered_gene_b"
- [readme] To generate cells and write them to a file, you can use the --generate flag: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers corresponding to the number of cells"
- [abstract] cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality: "cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality"
- [intro] we establish how a single-cell GAN (scGAN) can be leveraged to generate realistic scRNA-seq data: "we establish how a single-cell GAN (scGAN) can be leveraged to generate realistic scRNA-seq data"
