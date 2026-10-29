---
name: random-forest-discrimination-auc
description: 'Decision support for quantifying how realistic synthetic scRNA-seq cells
  are: train a random forest on real-vs-generated labels and use the resulting AUC,
  where values near 0.5 (positive control ~0.52) indicate generated cells are indistinguishable
  from real PBMC cells.'
when_to_use_negative:
- The goal is comparing distributional fidelity (MMD) rather than classifier-based
  realism — use an MMD-based comparison instead
- Generated cells were produced from a different training set than the real cells,
  since the discriminator must compare matched real and synthetic data
- The synthetic data is not single-cell RNA-seq count data from this scGAN/Splatter/SUGAR
  workflow
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scGAN
  role: Generates the synthetic PBMC cells whose realism is scored by the random forest
    discriminator
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant of the generator that can produce defined cell types for
    augmentation experiments
  repo: https://github.com/imsb-uke/scGAN
- name: SUGAR
  role: Baseline geometry-based generator (Synthesis Using Geometrically Aligned Random-walks)
    run on the same PBMC training cells; its generated cells yield RF-AUC 0.98
  repo: https://github.com/KrishnaswamyLab/SUGAR
- name: Splatter
  role: Baseline scRNA-seq simulator run on the same PBMC training cells as a comparison
    method
- name: Python
  role: Implements the random forest classifier and AUC computation for the real-vs-generated
    discrimination task
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:8e428673678abe41879d2afe235cf1ab098d78009b68b35da83719db532aba16
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# random-forest-discrimination-auc

## Summary

Train a random forest classifier to discriminate real scRNA-seq cells from GAN-generated cells and record the cross-validated AUC (RF-AUC) as a realism metric for synthetic data, benchmarked against Splatter and SUGAR on the Fresh 68k PBMCs (Donor A) dataset.

## When to use

Decision support for quantifying how realistic synthetic scRNA-seq cells are: train a random forest on real-vs-generated labels and use the resulting AUC, where values near 0.5 (positive control ~0.52) indicate generated cells are indistinguishable from real PBMC cells.

## When NOT to use

- The goal is comparing distributional fidelity (MMD) rather than classifier-based realism — use an MMD-based comparison instead
- Generated cells were produced from a different training set than the real cells, since the discriminator must compare matched real and synthetic data
- The synthetic data is not single-cell RNA-seq count data from this scGAN/Splatter/SUGAR workflow

## Inputs

- Real pre-processed Fresh 68k PBMCs (Donor A) training cells in .h5/.h5ad format
- Generated cells from scGAN (scgan_cells.h5ad), Splatter, and SUGAR matched to the real training data
- Cell x gene count matrices with real-vs-generated binary labels

## Outputs

- RF-AUC value per generation method (scGAN, Splatter, SUGAR) plus a real-data positive control
- Rows appended to mmd_rf_auc_benchmark.csv alongside MMD values, validated against Supplementary Tables 2-3

## How to apply

After generating matched synthetic sets with scGAN (via main.py --generate), Splatter, and SUGAR from the same pre-processed Fresh 68k PBMCs (Donor A) training cells stored in .h5/.h5ad, label each real cell and each generated cell and train a random forest classifier to distinguish real from generated for each method. Compute the AUC of this discriminator (RF-AUC); a low AUC near 0.5 means the classifier cannot separate real from synthetic, indicating high realism. Compute a real-data positive control by splitting the real PBMC cells themselves (expected AUC ~0.52). Compile MMD and RF-AUC per method into mmd_rf_auc_benchmark.csv and check against Supplementary Tables 2-3: Splatter MMD 129.52, SUGAR MMD 59.45 / AUC 0.98, scGAN MMD 0.87 / AUC 0.65.

## Related tools

- **scGAN** (Generates the synthetic PBMC cells whose realism is scored by the random forest discriminator) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant of the generator that can produce defined cell types for augmentation experiments) — https://github.com/imsb-uke/scGAN
- **SUGAR** (Baseline geometry-based generator (Synthesis Using Geometrically Aligned Random-walks) run on the same PBMC training cells; its generated cells yield RF-AUC 0.98) — https://github.com/KrishnaswamyLab/SUGAR
- **Splatter** (Baseline scRNA-seq simulator run on the same PBMC training cells as a comparison method)
- **Python** (Implements the random forest classifier and AUC computation for the real-vs-generated discrimination task)

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path scgan_cells.h5ad
```

## Evaluation signals

- Real-data positive control (discriminating real PBMC cells against real PBMC cells) yields RF-AUC ~0.52, close to the uninformative 0.5
- scGAN-generated cells yield RF-AUC ~0.65 while SUGAR yields ~0.98, matching Supplementary Tables 2-3
- Companion MMD values agree with the benchmark file: Splatter MMD 129.52, SUGAR MMD 59.45, scGAN MMD 0.87, real-data control MMD 0.03
- All method results compiled into mmd_rf_auc_benchmark.csv with real and generated cell counts matching (e.g. 1000/500/0/200 per cluster for scGAN generation)

## Limitations

- RF-AUC is a discriminator-based metric: an overly sensitive random forest can flag subtle artifacts, so it must be interpreted jointly with MMD and the real-data control AUC
- Scores depend on the exact pre-processing (filtering, clustering resolution, TFRecord split sizes) defined in parameters.; different settings change real/generated balance
- The paper's code was only tested with Python 3.5 and its packaged dependencies; newer scikit-learn/Python versions may alter random forest behavior and AUC reproducibility

## Evidence

- [abstract] cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality: "cscGAN outperforms existing methods for single-cell RNA-seq data generation in quality"
- [readme] generate cells and write them to a file: "To generate cells and write them to a file, you can use the `--generate` flag."
- [methods] most experiments use the Fresh 68k PBMCs (Donor A) dataset: "most of the experiments reported in our paper use the Fresh 68k PBMCs (Donor A) dataset"
- [supplementary] benchmark reference values from source card: [evidence span withheld — anchor status: not_found]
- [full_text] compile MMD and RF-AUC into benchmark CSV: [evidence span withheld — anchor status: not_found]
