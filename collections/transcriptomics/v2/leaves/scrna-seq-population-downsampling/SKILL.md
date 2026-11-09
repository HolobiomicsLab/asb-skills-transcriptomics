---
name: scrna-seq-population-downsampling
description: Use when designing a benchmark where a cell population is artificially
  sparsified to test whether trajectory or clustering outcomes depend on sampling
  density, before attempting generative restoration.
when_to_use_negative:
- The population is already too sparse to yield a meaningful downsampled subset or
  a stable original trajectory reference
- The goal is conditional generation of specific cell types from multi-type data —
  use cscGAN conditioning rather than plain downsampling plus unconditional scGAN
- The analysis goal does not involve testing sampling-density sensitivity (e.g. simple
  marker gene detection on the full population)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAN
  role: Unconditional generative model trained on the downsampled data to later generate
    replacement cells for the removed erythrocytes
  repo: https://github.com/imsb-uke/scGAN
- name: Python 3.5
  role: Runtime environment (main.py) for pre-processing the downsampled .h5/.h5ad
    data into TFRecord format
  repo: https://github.com/imsb-uke/scGAN
- name: Jupyter
  role: Provided notebook to convert source data to .h5/.h5ad before downsampling
    and pre-processing
  repo: https://github.com/imsb-uke/scGAN
- name: theislab/paga
  role: Used downstream to recompute the PAGA graph and force-directed layout on the
    downsampled vs. restored populations
  repo: https://github.com/theislab/paga
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:c7dcbd19f232bd2022b4d49f93c7222ce3d9382fd207dea719b66ea3d59bc839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Jupyter
  - Python
  - Python 3.5
  - Tensorboard
  - cscGAN
  - scGAN
  - theislab/paga
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrna-seq-population-downsampling

## Summary

Deliberately reduces a well-sampled scRNA-seq cell population (e.g. the 150-cell node-4 erythrocyte cluster from GSE72857 mouse bone marrow) to a sparse subset (e.g. 13 cells) to create a controlled test case for evaluating whether generative augmentation (scGAN) can restore downstream analyses such as PAGA pseudo-time trajectories.

## When to use

Use when designing a benchmark where a cell population is artificially sparsified to test whether trajectory or clustering outcomes depend on sampling density, before attempting generative restoration.

## When NOT to use

- The population is already too sparse to yield a meaningful downsampled subset or a stable original trajectory reference
- The goal is conditional generation of specific cell types from multi-type data — use cscGAN conditioning rather than plain downsampling plus unconditional scGAN
- The analysis goal does not involve testing sampling-density sensitivity (e.g. simple marker gene detection on the full population)

## Inputs

- Node-4 erythrocyte population (150 cells) extracted from GSE72857 mouse bone marrow scRNA-seq data
- Data in .h5 or .h5ad format (mtx/tsv convertible via the provided Jupyter notebook)
- parameters. configuration file (model type, layer sizes, normalization layer, lambda, batch size, latent space dimension, training/validation/test split)

## Outputs

- Downsampled population (13 cells) in .h5/.h5ad format ready for TFRecord pre-processing
- A degraded PAGA/force-directed trajectory baseline to compare against the original 150-cell graph

## How to apply

Extract the target population — e.g. the node-4 erythrocyte population of 150 cells from the GSE72857 mouse bone marrow dataset — keeping the data in .h5 or .h5ad format, since the scGAN pipeline requires .h5/.h5ad input (use the provided Jupyter notebook if converting from mtx/tsv). Randomly downsample the population from 150 to 13 cells while preserving the original file structure so that the original 150-cell analysis can serve as the reference. In the parameters. file, set the model type (scGAN for unconditional generation of the single cluster), the size of the layers, the normalization layer (batchnorm or layernorm), lambda, batch size, and latent space dimension, and set the unused test set size to 0. The downsampled data is then pre-processed into TFRecord format with the --process flag before scGAN training. The rationale is that PAGA topology inferred on the 13-cell population will degrade, providing a measurable baseline against which restoration by 137 scGAN-generated cells can be judged.

## Related tools

- **scGAN** (Unconditional generative model trained on the downsampled data to later generate replacement cells for the removed erythrocytes) — https://github.com/imsb-uke/scGAN
- **Python 3.5** (Runtime environment (main.py) for pre-processing the downsampled .h5/.h5ad data into TFRecord format) — https://github.com/imsb-uke/scGAN
- **Jupyter** (Provided notebook to convert source data to .h5/.h5ad before downsampling and pre-processing) — https://github.com/imsb-uke/scGAN
- **theislab/paga** (Used downstream to recompute the PAGA graph and force-directed layout on the downsampled vs. restored populations) — https://github.com/theislab/paga

## Examples

```
python main.py --param parameters. --process
```

## Evaluation signals

- Downsampled file retains the .h5/.h5ad schema and exactly 13 of the original 150 erythrocyte cells
- Pre-processing completes successfully into TFRecord format with test set size 0
- Original 150-cell PAGA/force-directed graph can be reproduced as the reference topology
- The 13-cell population produces a visibly degraded trajectory compared to Supplementary Fig. 17G-I, establishing the restoration baseline
- Tensorboard loss curves converge during subsequent scGAN training on the downsampled data

## Limitations

- The downsampled (13-cell) population alone cannot support meaningful trajectory inference, which is the point of the benchmark but makes it unsuitable for direct biological interpretation
- Random downsampling introduces stochasticity; different random subsets may degrade the trajectory differently
- The mechanism enabling generation from very small subpopulations is only speculated, not established, so downsampled training data may yield variable scGAN quality

## Evidence

- [other] workflow step 1 of source card: [evidence span withheld — anchor status: not_found]
- [readme] readme data format: "Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format. In case you want to use data in mtx/tsv, please use the provided Jupyter notebook to convert the data to `.h5` or `.h5ad`."
- [readme] readme test set: "The test set is not used. Unless you want to use a test set for further custom experiments, you can set its size to 0."
- [readme] readme parameters: "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the"
- [discussion] article discussion limitation: "It may be surprising or even suspicious that our cscGAN is able to learn to generate cells coming from very small subpopulations (e.g. 16 cells) so well."
