---
name: in-silico-cell-generation
description: Use when a scRNA-seq subpopulation is too sparse (e.g. 13 of 150 cells
  after downsampling) to support trajectory or downstream analyses, and realistic
  cells must be generated in silico to restore population structure.
when_to_use_negative:
- Data is not single-cell RNA-seq — application of cscGAN to other biomedical domains
  and data types (proteomics, whole genomes, imaging) is only speculative in the article
- The goal is conditioning on side information other than cell type (e.g. case/control
  samples) — this avenue is left untested as future work
- Input data is already sufficient in cell numbers and augmentation would not address
  sparsity of the target subpopulation
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAN
  role: Unconditional single-cell GAN used to generate realistic scRNA-seq cells to
    augment sparse populations
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional scGAN variant for generating cells of defined types or subpopulations
    on demand
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep-learning framework implementing the scGAN/cscGAN training and generation
    pipeline
  repo: https://github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitoring of generator/discriminator losses during training until convergence
- name: theislab/paga
  role: Recomputation of the PAGA graph and force-directed layout on the augmented
    population to validate restored trajectory structure
  repo: https://github.com/theislab/paga
- name: Docker
  role: Reproducible environment containing all dependencies via provided Dockerfile
    and ready-to-use image
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:7570a325821053be7ea2a9b00a4fe72708ea5c157a3ea984958ff4b8b7eaf66c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  - theislab/paga
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# in-silico-cell-generation

## Summary

Generates realistic single-cell RNA-seq cells in silico by training a single-cell GAN (scGAN/cscGAN, TensorFlow implementation from imsb-uke/scGAN) on .h5/.h5ad scRNA-seq data and sampling new cells to augment sparse populations, e.g. restoring a downsampled erythrocyte trajectory for PAGA analysis.

## When to use

Use when a scRNA-seq subpopulation is too sparse (e.g. 13 of 150 cells after downsampling) to support trajectory or downstream analyses, and realistic cells must be generated in silico to restore population structure.

## When NOT to use

- Data is not single-cell RNA-seq — application of cscGAN to other biomedical domains and data types (proteomics, whole genomes, imaging) is only speculative in the article
- The goal is conditioning on side information other than cell type (e.g. case/control samples) — this avenue is left untested as future work
- Input data is already sufficient in cell numbers and augmentation would not address sparsity of the target subpopulation

## Inputs

- Single-cell RNA-seq data in .h5 or .h5ad format (e.g. downsampled erythrocyte population from GSE72857 mouse bone marrow data)
- parameters. file with model hyperparameters (model type, layer sizes, normalization layer, lambda, batch size, latent space dimension)
- Trained scGAN/cscGAN model checkpoint (--model_path)
- Number of cells to generate per cluster (--cells_no)

## Outputs

- TFRecord-formatted preprocessed training/validation/test splits
- Trained scGAN/cscGAN model checkpoint and Tensorboard loss logs
- Generated cells saved as .h5ad file (e.g. generated_cells.h5ad)
- Augmented cell population and recomputed PAGA graph / force-directed layout

## How to apply

Convert data to .h5/.h5ad format, then configure the parameters. file with model type (scGAN or cscGAN), layer sizes, normalization layer (batchnorm or layernorm), lambda, batch size, and latent space dimension. Run pre-processing with 'python main.py --param parameters. --process' to convert the data into TFRecord format (set the unused test set size to 0). Train the model with 'python main.py --param parameters. --train', monitoring convergence of losses with Tensorboard. Generate the required number of cells per cluster with the --generate flag (e.g. --cells_no 137), specifying --model_path and --save_path to write an .h5ad file. Append generated cells to the real cells (13 real + 137 generated = original 150-cell erythrocyte population) and recompute downstream analyses such as the PAGA graph and force-directed layout.

## Related tools

- **scGAN** (Unconditional single-cell GAN used to generate realistic scRNA-seq cells to augment sparse populations) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional scGAN variant for generating cells of defined types or subpopulations on demand) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Deep-learning framework implementing the scGAN/cscGAN training and generation pipeline) — https://github.com/imsb-uke/scGAN
- **Tensorboard** (Monitoring of generator/discriminator losses during training until convergence)
- **theislab/paga** (Recomputation of the PAGA graph and force-directed layout on the augmented population to validate restored trajectory structure) — https://github.com/theislab/paga
- **Docker** (Reproducible environment containing all dependencies via provided Dockerfile and ready-to-use image) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 137 --model_path path/to/my/model --save_path generated_cells.h5ad
```

## Evaluation signals

- Generated .h5ad file exists at the specified --save_path with the requested number of cells (e.g. 137 cells for the erythrocyte cluster)
- t-SNE validation plots (computed on the whole validation set plus an equal number of generated cells) show generated cells overlapping the real cell distribution
- Tensorboard loss curves for generator and discriminator converge during --train
- PAGA graph and force-directed layout recomputed on the augmented 150-cell population reproduce the topology of Supplementary Fig. 17G-I, with generated cells occupying the erythrocyte node region
- Cell counts balance: generated + real cells restore the original population size (13 + 137 = 150 cells)

## Limitations

- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Conditioning on side information other than cell type (e.g. case/control samples) is untested and left for future work
- Applications beyond classification such as clustering, cell type detection, and data denoising are proposed but not demonstrated; validation of trajectory restoration is qualitative (comparison to published figure panels)

## Evidence

- [abstract] cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and generates realistic cells of defined types: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
- [methods] To run the pre-processing, run the main.py script with the --process flag: "To run the pre-processing, run the main.py script with the --process flag"
- [readme] To generate cells and write them to a file, you can use the --generate flag.: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers"
- [readme] you can use the Tensorboard to monitor the losses: "After or during the training, you can use the Tensorboard to monitor the losses (for instance) evolve."
- [readme] Single cell RNA-seq data has to be in the .h5 or .h5ad format: "Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format. In case you want to use data in mtx/tsv, please use the provided Jupyter notebook"
