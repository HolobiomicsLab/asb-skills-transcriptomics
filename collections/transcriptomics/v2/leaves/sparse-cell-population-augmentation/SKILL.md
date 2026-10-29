---
name: sparse-cell-population-augmentation
description: Use when a scRNA-seq subpopulation has been severely downsampled (e.g.
  150 erythrocyte cells reduced to 13) and you need to restore the original cell count
  so trajectory or graph analyses recover their structure.
when_to_use_negative:
- The population is not sparse — augmentation is aimed at sparse cell populations;
  well-sampled populations do not need synthetic cells and the article only demonstrates
  gains from augmenting sparse populations
- The goal requires condition-specific generation of defined cell types from multi-type
  data — that is the cscGAN use case, whereas this skill uses the non-conditional
  scGAN on an extracted single cluster
- You need to condition generation on side information other than cell type (e.g.
  case/control samples), which the authors explicitly leave untested for future work
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAN
  role: Non-conditional generative adversarial network trained on the downsampled
    population to generate replacement cells
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Backend implementing the scGAN training and generation
- name: Tensorboard
  role: Monitoring of loss evolution during scGAN training until convergence
- name: theislab/paga
  role: Recomputation of the PAGA graph and force-directed layout on the augmented
    population to validate restored topology
  repo: https://github.com/theislab/paga
- name: Python 3.5
  role: Runtime environment for the scGAN pipeline (tested with Python 3.5 and requirements.txt
    packages)
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:88db66e078064b30bba03c72b35098cfded82e12bacacae8f0c9524071e6d8e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Python
  - Python 3.5
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  - theislab/paga
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sparse-cell-population-augmentation

## Summary

Augment a downsampled or sparse scRNA-seq cell population by training a single-cell GAN (scGAN) on the retained cells, generating replacement cells with the trained model, and re-appending them to restore the original population size and downstream graph structure (e.g., a PAGA pseudo-time trajectory).

## When to use

Use when a scRNA-seq subpopulation has been severely downsampled (e.g. 150 erythrocyte cells reduced to 13) and you need to restore the original cell count so trajectory or graph analyses recover their structure.

## When NOT to use

- The population is not sparse — augmentation is aimed at sparse cell populations; well-sampled populations do not need synthetic cells and the article only demonstrates gains from augmenting sparse populations
- The goal requires condition-specific generation of defined cell types from multi-type data — that is the cscGAN use case, whereas this skill uses the non-conditional scGAN on an extracted single cluster
- You need to condition generation on side information other than cell type (e.g. case/control samples), which the authors explicitly leave untested for future work

## Inputs

- .h5 or .h5ad scRNA-seq count matrix containing the sparse/downsampled cell population (e.g. 13 of 150 erythrocyte cells from GSE72857 mouse bone marrow)
- parameters. file defining model type (scGAN), layer sizes, normalization layer, lambda, batch size, latent space dimension, and train/validation/test split sizes

## Outputs

- .h5ad file of generated cells (e.g. generated_cells.h5ad with 137 cells for the erythrocyte cluster)
- Restored full-size cell population (generated cells appended to retained real cells)
- Recomputed PAGA graph and force-directed layout on the augmented population

## How to apply

First convert the data to .h5 or .h5ad format (using the provided Jupyter notebook for mtx/tsv input) and configure parameters. with the model type (scGAN), layer sizes, normalization layer (batchnorm or layernorm), lambda, batch size, and latent space dimension, setting the test set size to 0. Run pre-processing with 'python main.py --param parameters. --process' to convert the data to TFRecord format, then train with '--train', monitoring loss convergence in Tensorboard. Generate the number of cells needed to restore the original population size with the '--generate' flag plus '--cells_no', '--model_path', and '--save_path', saving the synthetic cells as .h5ad. Append the generated cells to the retained real cells (e.g. 137 scGAN-generated + 13 real = 150 erythrocytes), then recompute downstream analyses such as the PAGA graph and force-directed layout (theislab/paga) on the augmented population and compare to the pre-downsampling graph.

## Related tools

- **scGAN** (Non-conditional generative adversarial network trained on the downsampled population to generate replacement cells) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Backend implementing the scGAN training and generation)
- **Tensorboard** (Monitoring of loss evolution during scGAN training until convergence)
- **theislab/paga** (Recomputation of the PAGA graph and force-directed layout on the augmented population to validate restored topology) — https://github.com/theislab/paga
- **Python 3.5** (Runtime environment for the scGAN pipeline (tested with Python 3.5 and requirements.txt packages)) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 137 --model_path path/to/trained/model --save_path generated_cells.h5ad
```

## Evaluation signals

- The generated .h5ad file exists and contains exactly the requested number of cells (--cells_no, e.g. 137), matching the removed cells so total population size is restored (137 + 13 = 150)
- The recomputed PAGA force-directed graph reproduces the topology of the original pre-downsampling graph (Supplementary Fig. 17G-I), with generated cells occupying the erythrocyte node region
- Training losses converge, as monitored via Tensorboard summary operations
- t-SNE validation plots (computed on the validation set plus an equal number of generated cells) show generated cells overlapping the real cell distribution

## Limitations

- The mechanism enabling the GAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established — training on extremely small retained sets may be unreliable
- The test set is not used by the pipeline; it should be set to 0 unless needed for custom experiments
- Validation of trajectory restoration is qualitative (comparison to published figure panels) rather than quantitative in the reproduced task
- Applications of augmentation beyond classification (clustering, cell type detection, data denoising) are proposed but not demonstrated by the authors

## Evidence

- [methods] To run the pre-processing, run the main.py script with the --process flag: "To run the pre-processing, run the main.py script with the --process flag"
- [methods] To generate cells and write them to a file, you can use the --generate flag.: "To generate cells and write them to a file, you can use the --generate flag."
- [readme] Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format.: "Single cell RNA-seq data has to be in the `.h5` or `.h5ad` format."
- [intro] our models can successfully augment sparse cell populations: "our models can successfully augment sparse cell populations to improve the quality and robustness of downstream classification"
- [readme] The type of model (scGAN or cscGAN): "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the"
