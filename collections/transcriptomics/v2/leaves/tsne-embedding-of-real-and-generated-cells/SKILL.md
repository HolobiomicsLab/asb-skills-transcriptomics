---
name: tsne-embedding-of-real-and-generated-cells
description: 'Use when validating scGAN or cscGAN generated single-cell RNA-seq data:
  embeddings must show generated cells overlapping real validation clusters and recapitulating
  cluster/marker-gene structure.'
when_to_use_negative:
- No held-out validation set exists because the test set size was set to 0 and no
  validation split was retained — there are no real cells to co-embed
- The goal is quantitative realism assessment (e.g. Random Forest AUC 0.62/0.55 or
  MMD 0.286/0.238 for PBMC clusters 2 and 6) rather than visual cluster/marker-gene
  validation
- The input is a dataset format not yet converted to .h5/.h5ad (e.g. raw mtx/tsv),
  which must first be converted with the provided Jupyter notebook
edam_operation: http://edamontology.org/operation_0564
tools:
- name: t-SNE
  role: Computes the joint low-dimensional embedding of the whole validation set and
    an equal number of generated cells for the validation plots
- name: scGAN
  role: Provides the generated cells and validation set; training emits the non-conditional
    t-SNE validation plots at a configured frequency
  repo: github.com/imsb-uke/scGAN
- name: cscGAN
  role: Generates cluster-conditioned cells whose overlap with target PBMC clusters
    is assessed via the conditional t-SNE plot
  repo: github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Companion monitoring of generator/discriminator losses alongside the periodic
    t-SNE validation plots during training
  repo: github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Underlying framework in which the scGAN/cscGAN training loop computes and
    renders the t-SNE validation plots
  repo: github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:a4f081746f83ad25f4e8ddc4a57eb9688027118b8daa2161e15eef306ceaf026
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
  - scGAN
  - t-SNE
  edam_operation: http://edamontology.org/operation_0564
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tsne-embedding-of-real-and-generated-cells

## Summary

Produces t-SNE validation plots in the scGAN/cscGAN pipeline by jointly embedding the whole validation set together with an equal number of GAN-generated scRNA-seq cells, to visually confirm that generated cells overlap real clusters and reproduce marker-gene structure.

## When to use

Use when validating scGAN or cscGAN generated single-cell RNA-seq data: embeddings must show generated cells overlapping real validation clusters and recapitulating cluster/marker-gene structure.

## When NOT to use

- No held-out validation set exists because the test set size was set to 0 and no validation split was retained — there are no real cells to co-embed
- The goal is quantitative realism assessment (e.g. Random Forest AUC 0.62/0.55 or MMD 0.286/0.238 for PBMC clusters 2 and 6) rather than visual cluster/marker-gene validation
- The input is a dataset format not yet converted to .h5/.h5ad (e.g. raw mtx/tsv), which must first be converted with the provided Jupyter notebook

## Inputs

- Validation-set scRNA-seq data in TFRecord format produced by the --process pre-processing step from .h5/.h5ad input
- Generated cells from a trained scGAN/cscGAN model (equal in number to the validation set), optionally saved as .h5ad
- parameters. with plotting frequency, model path, and experiment directory settings

## Outputs

- Non-conditional t-SNE validation plot embedding whole validation set plus equal number of generated cells
- Conditional t-SNE plot showing generated cells overlapping their target clusters (cscGAN)

## How to apply

During (c)scGAN training, the pipeline periodically computes each t-SNE embedding using the whole validation set plus an equal number of generated cells, with the plotting frequency configured in the parameters. file alongside the Tensorboard logging and checkpoint-saving frequencies. For unconditional scGAN, the plot shows all validation cells and generated cells mixed; for cscGAN, the conditional t-SNE should show generated cells overlapping their target PBMC clusters. The validation set must be held out during preprocessing ('python main.py --param parameters. --process' splits the .h5/.h5ad data into TFRecords), and generation can be run separately with the --generate and --cells_no flags saving to .h5ad. Acceptance criterion: generated-cell t-SNE embedding overlaps the real validation-set clusters and represents marker-gene structure, as illustrated by the reference non-conditional and conditional t-SNE images in the repository.

## Related tools

- **t-SNE** (Computes the joint low-dimensional embedding of the whole validation set and an equal number of generated cells for the validation plots)
- **scGAN** (Provides the generated cells and validation set; training emits the non-conditional t-SNE validation plots at a configured frequency) — github.com/imsb-uke/scGAN
- **cscGAN** (Generates cluster-conditioned cells whose overlap with target PBMC clusters is assessed via the conditional t-SNE plot) — github.com/imsb-uke/scGAN
- **Tensorboard** (Companion monitoring of generator/discriminator losses alongside the periodic t-SNE validation plots during training) — github.com/imsb-uke/scGAN
- **TensorFlow** (Underlying framework in which the scGAN/cscGAN training loop computes and renders the t-SNE validation plots) — github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --train
```

## Evaluation signals

- Generated-cell t-SNE embedding overlaps the real validation-set clusters (compare against the reference non-cond_t-SNE.jpg and cond_t-SNE.jpg images in the repository)
- Generated cells reproduce marker-gene structure visible in the t-SNE embedding of the validation set
- The number of generated cells embedded equals the size of the whole validation set (equal-number co-embedding invariant)
- Conditional t-SNE shows cscGAN-generated cells of a given cluster positioned within their target real cluster
- Plots are regenerated at the configured t-SNE plotting frequency declared in parameters.

## Limitations

- t-SNE is only a visual validation signal; the paper's quantitative realism metrics (Random Forest AUC, MMD) are separate and should supplement the plots
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, so small-subpopulation t-SNE overlap should be interpreted cautiously
- Embedding quality depends on the pre-processing split: the validation set (not the test set, which is typically set to size 0) is what is co-embedded

## Evidence

- [readme] Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells).: "Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells)."
- [methods] the validation set is used to produce the t-SNE plots: "the validation set is used to produce the t-SNE plots"
- [discussion] cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells.: "cscGAN outperforms current methods in the realistic generation of scRNA-seq data and scales sublinearly in the number of cells."
