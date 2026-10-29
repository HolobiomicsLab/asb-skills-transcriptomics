---
name: cell-cycle-phase-inference-deep-learning
description: Use when scRNA-seq data contains spliced/unspliced layers, cells are
  proliferating, and a continuous cell cycle phase assignment is needed from RNA velocity
  dynamics rather than gene-marker scoring.
when_to_use_negative:
- No cycling genes are detected in the dataset — the README states results will not
  be reliable and deeper sequencing is needed
- The input Anndata lacks spliced/unspliced layers or has not been preprocessed with
  scvelo.pp.moments
- The population of interest is non-proliferating or quiescent with no cell cycle
  cycling patterns
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Deep learning autoencoder that fits cycling patterns in unspliced-spliced
    RNA space to assign a continuous cell cycle theta to each cell
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Computes the imputed moments (Ms, Mu layers) required as preprocessing before
    DeepCycle inference
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Generates the spliced and unspliced count matrices from the scRNA-seq alignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:6786d39e40bf17fa7ce57a16dc87ab526e08597a2bf1835e12c50be0b2d5d228
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - TensorFlow
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cell-cycle-phase-inference-deep-learning

## Summary

DeepCycle assigns a continuous cell cycle angle (theta) to single cells by fitting unspliced-spliced RNA velocity patterns with an autoencoder, producing a high-resolution map of the entire cell cycle transcriptome without synchronization or fluorescent reporters.

## When to use

Use when scRNA-seq data contains spliced/unspliced layers, cells are proliferating, and a continuous cell cycle phase assignment is needed from RNA velocity dynamics rather than gene-marker scoring.

## When NOT to use

- No cycling genes are detected in the dataset — the README states results will not be reliable and deeper sequencing is needed
- The input Anndata lacks spliced/unspliced layers or has not been preprocessed with scvelo.pp.moments
- The population of interest is non-proliferating or quiescent with no cell cycle cycling patterns

## Inputs

- Anndata file preprocessed with velocyto and scvelo.pp.moments (contains 'Ms' and 'Mu' layers)
- Gene list file (e.g., GOterm cell_cycle annotation for the organism of interest)
- Base gene name (e.g., MELK for human fibroblasts, Nusap1 for mESCs)
- Expression threshold value (suggested around 1.0)

## Outputs

- Output Anndata with adata.obs['cell_cycle_theta'] assigned to each cell
- ./fits/ directory with unspliced-spliced pattern plots
- ./training/ directory with MSE plots during training
- ./DeepCycle_model/ TensorFlow model directory
- input_data.npz (input data used to train the model)
- ./hotelling/ directory of genes showing more than one state of expression (only with --hotelling)

## How to apply

Preprocess the Anndata with velocyto and scvelo.pp.moments to generate imputed Mu and Ms layers, then subset the proliferating subpopulation (e.g., leiden cluster 0 for human fibroblasts). Select a base gene as the initial condition for the transcriptional phase—MELK for human fibroblasts, Nusap1 for mESCs—and supply a GOterm cell_cycle gene list. Run DeepCycle.py with --expression_threshold around 1.0 (adjusted to sequencing depth), optionally enabling --gpu and --hotelling to identify cycling genes showing at least two expression states. Inspect ./fits/ unspliced-spliced pattern plots and ./training/ MSE plots to confirm the autoencoder (trained with Adam on MSE, early stopping with patience 20, batch size 5, 17% validation) fit correctly. Extract adata.obs['cell_cycle_theta'] from the output Anndata; if no cycling genes are found, the results are unreliable and deeper sequencing is required.

## Related tools

- **DeepCycle** (Deep learning autoencoder that fits cycling patterns in unspliced-spliced RNA space to assign a continuous cell cycle theta to each cell) — https://github.com/andreariba/DeepCycle
- **scVelo** (Computes the imputed moments (Ms, Mu layers) required as preprocessing before DeepCycle inference) — https://scvelo.readthedocs.io/
- **velocyto** (Generates the spliced and unspliced count matrices from the scRNA-seq alignment)

## Examples

```
python DeepCycle.py --input_adata fibroblast_leiden0.h5ad --gene_list GO_cell_cycle_annotation_human.txt --base_gene MELK --expression_threshold 1.0 --gpu --hotelling --output_adata fibroblast_deepcycle_annotated.h5ad
```

## Evaluation signals

- Cycling genes are identified in the output; absence of cycling genes means the results are not reliable
- ./fits/ unspliced-spliced pattern plots show proper fitting of the deep learning model to the cycling gene dynamics
- ./training/ MSE plots show convergence with early stopping (patience 20) and no overfitting
- adata.obs['cell_cycle_theta'] is populated for all cells in the output Anndata
- estimate_cell_cycle_transitions.py successfully estimates thetas for G1/S, S/G2 and M/G1 transitions from the output

## Limitations

- If no cycling genes are found, the results will not be reliable; deeper sequencing and more RNAs per cell may be required
- The current version requires a mandatory preprocessing step with scvelo.pp.moments
- Defining cell cycle phases based solely on gene markers lacks generalizability across cellular models; expression threshold must be adjusted to sequencing depth per cell

## Evidence

- [abstract] we propose a deep learning approach (DeepCycle) to fit these patterns and build a high-resolution map of the entire cell cycle transcriptome: "we propose a deep learning approach (DeepCycle) to fit these patterns and build a high-resolution map of the entire cell cycle transcriptome"
- [readme] The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [readme] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [intro] we designed DeepCycle (https://github.com/andreariba/DeepCycle), a deep learning method to ascribe a continuous high-resolution cell cycle trajectory to single cells based on RNA velocity: "we designed DeepCycle (https://github.com/andreariba/DeepCycle), a deep learning method to ascribe a continuous high-resolution cell cycle trajectory to single cells based on RNA velocity"
- [readme] If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
