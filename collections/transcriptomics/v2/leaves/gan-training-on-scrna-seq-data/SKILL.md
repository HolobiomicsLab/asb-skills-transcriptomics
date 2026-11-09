---
name: gan-training-on-scrna-seq-data
description: Use when a scRNA-seq population (e.g. an erythrocyte cluster of 150 cells
  in GSE72857 bone marrow data) has been downsampled or is too sparse, and you need
  to generate matched synthetic cells in .h5ad format to restore pseudo-time/PAGA
  graph structure or improve classifier robustness.
when_to_use_negative:
- Input data is in raw mtx/tsv format and has not yet been converted to .h5/.h5ad
  via the provided Jupyter notebook
- The goal is validation/comparison against a held-out test set and you intend to
  use the test split — the scGAN pipeline explicitly does not use the test set, so
  its size should be set to 0
- The downstream task requires conditional generation of defined cell types from complex
  multi-type samples and you only have an unconditional model — cscGAN is required
  for on-demand cell-type-specific generation
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scGAN
  role: Unconditional single-cell GAN model trained on the scRNA-seq data to generate
    realistic synthetic cells; selected via the 'type of model' parameter in parameters.
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant used to generate cells of defined cell types or subpopulations
    on demand (e.g. matched erythrocyte cells)
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Framework implementing the scGAN/cscGAN training and generation; Docker image
    provided for environment reproducibility
  repo: https://github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Used to monitor training losses until convergence via summary operations defined
    in the model
  repo: https://github.com/imsb-uke/scGAN
- name: paga
  role: Downstream tool used to recompute the PAGA graph and force-directed layout
    on the augmented population to validate restored trajectory structure
  repo: https://github.com/theislab/paga
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:c9d5e3e6bf3c09b32d4ccda8316988b9fe9450d6dc2b3d2a0c77c348c0067c00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - paga
  - scGAN
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# gan-training-on-scrna-seq-data

## Summary

Train a single-cell generative adversarial network (scGAN/cscGAN) on .h5/.h5ad scRNA-seq data using the TensorFlow implementation from Marouf et al. 2020, to generate realistic in silico cells for augmenting sparse cell populations and restoring downstream structures such as PAGA trajectories.

## When to use

Use when a scRNA-seq population (e.g. an erythrocyte cluster of 150 cells in GSE72857 bone marrow data) has been downsampled or is too sparse, and you need to generate matched synthetic cells in .h5ad format to restore pseudo-time/PAGA graph structure or improve classifier robustness.

## When NOT to use

- Input data is in raw mtx/tsv format and has not yet been converted to .h5/.h5ad via the provided Jupyter notebook
- The goal is validation/comparison against a held-out test set and you intend to use the test split — the scGAN pipeline explicitly does not use the test set, so its size should be set to 0
- The downstream task requires conditional generation of defined cell types from complex multi-type samples and you only have an unconditional model — cscGAN is required for on-demand cell-type-specific generation

## Inputs

- Single-cell RNA-seq data in .h5 or .h5ad format (e.g. downsampled erythrocyte population of 13 cells from GSE72857 mouse bone marrow)
- parameters. configuration file (model type, layer sizes, normalization layer, lambda, batch size, latent space dimension, dataset paths, train/validation/test set sizes)
- mtx/tsv count matrices convertible via the provided Jupyter notebook

## Outputs

- Trained scGAN/cscGAN model checkpoint (path specified in parameters.)
- Generated cells saved to .h5ad file (e.g. generated_cells.h5ad with 137 cells to augment 13 real cells back to 150)
- Tensorboard loss logs and t-SNE validation plots comparing real vs. generated cells

## How to apply

First convert data to .h5 or .h5ad format (use the provided Jupyter notebook for mtx/tsv inputs). Configure model (hyper-)parameters in a parameters. file — model type (scGAN or cscGAN), layer sizes, normalization layer (batchnorm or layernorm), lambda, batch size, latent space dimension, and train/validation/test set sizes (set the test set size to 0 unless needed for custom experiments). Run pre-processing with `python main.py --param parameters. --process` to convert data to TFRecord format for performance, then train with `python main.py --param parameters. --train`, monitoring loss convergence with Tensorboard (t-SNE validation plots against the validation set are produced automatically). Once converged, generate the required number of cells per cluster with the --generate, --cells_no, --model_path, and --save_path flags; append generated cells to the remaining real cells to restore the original population size, then recompute downstream analyses (e.g. PAGA graph and force-directed layout via theislab/paga) on the augmented population.

## Related tools

- **scGAN** (Unconditional single-cell GAN model trained on the scRNA-seq data to generate realistic synthetic cells; selected via the 'type of model' parameter in parameters.) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant used to generate cells of defined cell types or subpopulations on demand (e.g. matched erythrocyte cells)) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Framework implementing the scGAN/cscGAN training and generation; Docker image provided for environment reproducibility) — https://github.com/imsb-uke/scGAN
- **Tensorboard** (Used to monitor training losses until convergence via summary operations defined in the model) — https://github.com/imsb-uke/scGAN
- **paga** (Downstream tool used to recompute the PAGA graph and force-directed layout on the augmented population to validate restored trajectory structure) — https://github.com/theislab/paga

## Examples

```
python main.py --param parameters. --generate --cells_no 137 --model_path path/to/my/model --save_path generated_cells.h5ad
```

## Evaluation signals

- t-SNE validation plots (computed on the whole validation set plus an equal number of generated cells) show generated cells overlapping the real cell distribution
- Tensorboard losses (generator/discriminator) converge and stabilize during training
- File existence of the generated .h5ad output and the trained model checkpoint at --save_path/--model_path
- Recomputed PAGA graph and force-directed layout on the augmented population reproduces the topology of the original (pre-downsampling) trajectory, with generated cells occupying the erythrocyte node region (cf. Supplementary Fig. 17G-I)
- Generated + real cell count matches the original population size (e.g. 13 real + 137 generated = 150 erythrocyte cells)

## Limitations

- The test set is not used by the pipeline; only the validation set is used for t-SNE plots, so held-out quantitative evaluation requires custom experiments
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Conditioning on side information other than cell type (e.g. case/control samples) is untested and left for future work
- Training must be monitored to convergence via Tensorboard losses; the code was tested with Python 3.5 and dependencies are assumed covered by requirements.txt or the provided Docker image

## Evidence

- [abstract] we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data: "we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data"
- [methods] To run the pre-processing, run the main.py script with the --process flag: "To run the pre-processing, run the main.py script with the --process flag"
- [methods] the validation set is used to produce the t-SNE plots: "the validation set is used to produce the t-SNE plots"
- [readme] For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format: "For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format, which in turn will be used for the training of the model."
- [readme] you can use the Tensorboard to monitor the losses (for instance) evolve.: "After or during the training, you can use the Tensorboard to monitor the losses (for instance) evolve."
