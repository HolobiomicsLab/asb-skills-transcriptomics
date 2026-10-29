---
name: gan-training-on-single-cell-data
description: Use when scRNA-seq data (.h5/.h5ad, e.g. Fresh 68k PBMCs Donor A or a
  1.3M-cell mouse brain dataset) must be augmented or realistic cells must be generated
  in silico, either unconditionally or conditioned on cell type.
when_to_use_negative:
- The goal is manifold-based or graph-based data generation rather than adversarial
  training — use alternatives like KrishnaswamyLab/SUGAR, which generates points along
  a learned diffusion-geometry manifold
- The task is trajectory/connectivity inference rather than cell generation — PAGA
  (partition-based graph abstraction) is the appropriate tool
- Data cannot be converted to .h5/.h5ad or the pipeline cannot be run in the Docker/TensorFlow
  environment
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAN
  role: Unconditional single-cell GAN trained to generate realistic scRNA-seq data
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant of scGAN used to generate cells of defined types/subpopulations
    on demand via conditioning
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep learning framework implementing and training the (c)scGAN models
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Reproducible environment containing all dependencies, built via dockerfile/Dockerfile
  repo: https://github.com/imsb-uke/scGAN
- name: Jupyter
  role: Notebook provided to convert mtx/tsv data to .h5 or .h5ad before preprocessing
- name: Tensorboard
  role: Monitoring generator/discriminator loss evolution during training
- name: t-SNE
  role: Validation embedding of the whole validation set plus an equal number of generated
    cells
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:34364b986447eed6bb49480ebd14953a36ef3a62916928140a17cccfc8b18177
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# gan-training-on-single-cell-data

## Summary

Train and use the scGAN/cscGAN TensorFlow models from Marouf et al. (Nature Communications 2020) to generate realistic scRNA-seq data, including conditional generation of defined cell types and augmentation of large datasets such as the 1.3M-cell mouse brain dataset.

## When to use

Use when scRNA-seq data (.h5/.h5ad, e.g. Fresh 68k PBMCs Donor A or a 1.3M-cell mouse brain dataset) must be augmented or realistic cells must be generated in silico, either unconditionally or conditioned on cell type.

## When NOT to use

- The goal is manifold-based or graph-based data generation rather than adversarial training — use alternatives like KrishnaswamyLab/SUGAR, which generates points along a learned diffusion-geometry manifold
- The task is trajectory/connectivity inference rather than cell generation — PAGA (partition-based graph abstraction) is the appropriate tool
- Data cannot be converted to .h5/.h5ad or the pipeline cannot be run in the Docker/TensorFlow environment

## Inputs

- scRNA-seq count matrix in .h5 or .h5ad format (or mtx/tsv converted via the provided Jupyter notebook)
- parameters. parameter file with preprocessing, model, and training hyperparameters
- Per-cluster cell counts for generation (e.g. via --cells_no)

## Outputs

- TFRecord-formatted preprocessed training/validation/test splits
- Trained scGAN/cscGAN model checkpoints and Tensorboard loss logs
- Generated single-cell expression matrix saved as .h5ad
- t-SNE validation plots comparing validation-set and generated cells

## How to apply

Convert input scRNA-seq data to .h5 or .h5ad using the provided Jupyter notebook if it arrives as mtx/tsv, then define filtering parameters, dataset path, training/validation/test split sizes (test set size 0 unless needed), and clustering resolution in parameters. and run preprocessing with 'python main.py --param parameters. --process' to store splits in TFRecord format. Configure model type (scGAN or cscGAN), layer sizes, normalization layer (batchnorm or layernorm), lambda, batch size, latent space dimension, and optimizer parameters in the same JSON file, then train with 'python main.py --param parameters. --train' on a single GPU (a list of GPU identifiers 0..n is given in the parameter file), resuming from a checkpoint path if needed. Monitor generator/discriminator losses in Tensorboard during training; the validation set is used to produce t-SNE plots embedding the whole validation set together with an equal number of generated cells. For conditional generation with cscGAN, supply per-cluster cell counts via '--cells_no' (e.g. '1000 500 0 200') and save output to an .h5ad file. The rationale is that cscGAN learns non-linear gene-gene dependencies from complex multi-cell-type samples and scales sublinearly in the number of cells, so it can reproduce marker-gene structure of trained clusters even on million-cell datasets.

## Related tools

- **scGAN** (Unconditional single-cell GAN trained to generate realistic scRNA-seq data) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant of scGAN used to generate cells of defined types/subpopulations on demand via conditioning) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Deep learning framework implementing and training the (c)scGAN models) — https://github.com/imsb-uke/scGAN
- **Docker** (Reproducible environment containing all dependencies, built via dockerfile/Dockerfile) — https://github.com/imsb-uke/scGAN
- **Jupyter** (Notebook provided to convert mtx/tsv data to .h5 or .h5ad before preprocessing)
- **Tensorboard** (Monitoring generator/discriminator loss evolution during training)
- **t-SNE** (Validation embedding of the whole validation set plus an equal number of generated cells)

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- Generated-cell t-SNE embedding overlaps the real validation-set clusters and reproduces marker-gene structure
- Generator/discriminator losses in Tensorboard converge without divergence over training steps
- Conditional cscGAN generates cells representing every trained cluster with accurately learned marker gene expression patterns
- Wall-clock training time scales sublinearly (at worst linearly) with the number of cells on the million-cell dataset
- Generated .h5ad file contains the requested per-cluster cell counts specified via --cells_no

## Limitations

- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Conditioning on side information other than cell type (e.g. case/control samples) is untested and left for future work
- Applications beyond classification (clustering, cell type detection, denoising) and extension to other data types (proteomics, whole genomes, imaging) are proposed or speculated but not demonstrated
- The test set is not used by the pipeline unless configured for custom experiments; set its size to 0 otherwise

## Evidence

- [abstract] cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and generates realistic cells of defined types: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
- [discussion] cscGAN scales sublinearly in number of cells: "cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells."
- [readme] Data must be .h5/.h5ad; convert mtx/tsv with Jupyter notebook: "Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format. In case you want to use data in mtx/tsv, please use the provided Jupyter notebook to convert"
- [readme] Preprocessing stores splits in TFRecord; run with --process: "For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format, which in turn will be used for the training of the model."
- [methods] Model hyperparameters defined in JSON and training run with --train: "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the"
- [readme] Generation uses --generate with --cells_no and saves output: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers corresponding to the number of cells"
- [other] Trained on 1.3M mouse brain cells with sublinear runtime scaling: [evidence span withheld — anchor status: not_found]
