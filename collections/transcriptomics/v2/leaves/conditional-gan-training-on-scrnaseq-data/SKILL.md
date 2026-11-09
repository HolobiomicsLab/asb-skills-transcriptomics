---
name: conditional-gan-training-on-scrnaseq-data
description: Use when the goal is to generate realistic scRNA-seq cells of defined
  cell types or subpopulations on demand (e.g., augmenting sparse clusters like PBMC
  clusters 2 and 6) from cluster-annotated .h5/.h5ad data.
when_to_use_negative:
- Input data is not in .h5/.h5ad and has not been converted (mtx/tsv must first be
  converted with the provided Jupyter notebook)
- The task is unconditional generation of cells without cell-type conditioning — the
  plain scGAN model type, not cscGAN, is the appropriate setting
- The goal is conditioning on side information other than cell type (e.g., case/control
  samples) — the article explicitly leaves this avenue untested for future work
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: cscGAN
  role: Conditional GAN model trained to generate cells of specified clusters on demand;
    implemented in TensorFlow in imsb-uke/scGAN
  repo: github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep-learning backend in which the cscGAN adversarial training and generation
    run (provided as a ready-to-use Docker image)
  repo: github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitors the evolution of generator/discriminator losses and summary quantities
    during training
  repo: github.com/imsb-uke/scGAN
- name: t-SNE
  role: Produces validation plots embedding the whole validation set with an equal
    number of generated cells to visually check conditional realism
  repo: github.com/imsb-uke/scGAN
- name: Docker
  role: Reproducible environment with all dependencies, built via dockerfile/Dockerfile
  repo: github.com/imsb-uke/scGAN
- name: Jupyter
  role: Provided notebook converts mtx/tsv input data to .h5 or .h5ad before pre-processing
  repo: github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:8ba865daa42489861b1631d7cdc56dee814e50606f8a88fa8f8c867fb48c93d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# conditional-gan-training-on-scrnaseq-data

## Summary

Train a conditional single-cell GAN (cscGAN) with TensorFlow on cluster-annotated scRNA-seq data so that realistic cells of specific types (e.g., PBMC clusters) can be generated on demand, following the Marouf et al. 2020 Nature Communications workflow implemented in imsb-uke/scGAN.

## When to use

Use when the goal is to generate realistic scRNA-seq cells of defined cell types or subpopulations on demand (e.g., augmenting sparse clusters like PBMC clusters 2 and 6) from cluster-annotated .h5/.h5ad data.

## When NOT to use

- Input data is not in .h5/.h5ad and has not been converted (mtx/tsv must first be converted with the provided Jupyter notebook)
- The task is unconditional generation of cells without cell-type conditioning — the plain scGAN model type, not cscGAN, is the appropriate setting
- The goal is conditioning on side information other than cell type (e.g., case/control samples) — the article explicitly leaves this avenue untested for future work

## Inputs

- Cluster-annotated single-cell RNA-seq data in .h5 or .h5ad format (e.g., Fresh 68k PBMCs Donor A; mtx/tsv convertible via the provided Jupyter notebook)
- parameters. configuration file (model type, layer sizes, normalization layer, lambda, batch size, latent dimension, optimizer settings, GPU ids, paths, filtering parameters, train/validation/test sizes, clustering resolution)

## Outputs

- Trained cscGAN model checkpoint
- Generated cluster-conditioned cells written to a .h5ad file (e.g., generated_cells.h5ad)
- Tensorboard loss curves and periodic conditional t-SNE validation plots

## How to apply

Fill in the parameters. template setting the model type to cscGAN, choose the batchnorm normalization layer for conditioning (vs layernorm), and specify layer sizes, lambda, batch size, latent-space dimension, and optimizer settings (steps, learning rate, algorithm), plus GPU identifiers, experiment name, and directory paths. Place cluster-annotated data in .h5 or .h5ad format (convert mtx/tsv with the provided Jupyter notebook; the Fresh 68k PBMCs Donor A dataset is the reference example), define filtering parameters, training/validation/test set sizes (test size can be 0) and clustering resolution, then run pre-processing with 'python main.py --param parameters. --process' to produce TFRecord files. Launch adversarial training with '--train' on a single GPU, optionally resuming from a checkpoint path in the . file, monitoring generator/discriminator losses in Tensorboard and periodic conditional t-SNE plots that embed the whole validation set alongside an equal number of generated cells. Generate cluster-conditioned cells with '--generate --cells_no <ints per cluster> --model_path ... --save_path *.h5ad'. Judge success by a Random Forest classifier trained to distinguish generated from real cells of the target clusters (higher AUC = more distinguishable; reference cscGAN reached only AUC 0.62 for cluster 2 and 0.55 for cluster 6) plus MMD distances (reference: 0.286 and 0.238), and by overlap of generated cells with target clusters in the conditional t-SNE.

## Related tools

- **cscGAN** (Conditional GAN model trained to generate cells of specified clusters on demand; implemented in TensorFlow in imsb-uke/scGAN) — github.com/imsb-uke/scGAN
- **TensorFlow** (Deep-learning backend in which the cscGAN adversarial training and generation run (provided as a ready-to-use Docker image)) — github.com/imsb-uke/scGAN
- **Tensorboard** (Monitors the evolution of generator/discriminator losses and summary quantities during training) — github.com/imsb-uke/scGAN
- **t-SNE** (Produces validation plots embedding the whole validation set with an equal number of generated cells to visually check conditional realism) — github.com/imsb-uke/scGAN
- **Docker** (Reproducible environment with all dependencies, built via dockerfile/Dockerfile) — github.com/imsb-uke/scGAN
- **Jupyter** (Provided notebook converts mtx/tsv input data to .h5 or .h5ad before pre-processing) — github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 --model_path path/to/model --save_path generated_cells.h5ad
```

## Evaluation signals

- Random Forest classifier AUC distinguishing generated from real cells of the target cluster stays near chance (reference: AUC 0.62 for PBMC cluster 2, 0.55 for cluster 6 — higher AUC means the fake cells are easier to detect)
- Maximum mean discrepancy (MMD) between generated and real cells of each target cluster approaches reference values (0.286 for cluster 2, 0.238 for cluster 6)
- Conditional t-SNE shows generated cells overlapping their target clusters when embedded with the whole validation set
- Generator and discriminator losses in Tensorboard evolve stably without mode collapse during --train
- Generation command completes and writes the requested number of cells per cluster (one int per cluster index via --cells_no) to the specified .h5ad save path

## Limitations

- Each model is trained on a single GPU; training a separate model per experiment configuration is required
- The test set is not used by the pipeline unless set aside for custom experiments (its size can be set to 0)
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g., 16 cells) is only speculated, not established
- Conditioning on side information other than cell type (e.g., case/control samples) is untested and left for future work
- Data was tested with Python 3.5 and the pinned requirements; use the provided Dockerfile for a working dependency environment

## Evidence

- [abstract] cscGAN  [section=abstract: "we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data"
- [methods] cscGAN definition: "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda"
- [methods] TFRecord pre-processing: "To run the pre-processing, run the main.py script with the --process flag"
- [methods] checkpoint resume: "you can also resume the training by specifying the path to a checkpoint in the . file."
- [methods] t-SNE validation: "the validation set is used to produce the t-SNE plots"
- [readme] README generation example: [evidence span withheld — anchor status: not_found]
- [readme] README Docker: "You can build the docker image using `docker build . -t scgan:latest -f dockerfile/Dockerfile` ."
- [readme] README dataset: "most of the experiments reported in our paper use the [Fresh 68k PBMCs (Donor A) dataset]"
- [discussion] small subpopulation caveat: "It may be surprising or even suspicious that our cscGAN is able to learn to generate cells coming from very small subpopulations (e.g. 16 cells) so well."
- [discussion] scaling claim: "cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells."
