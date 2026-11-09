---
name: unspliced-spliced-dynamics-fitting
description: Use when a velocyto/scVelo-processed scRNA-seq dataset (h5ad with Mu/Ms
  layers) shows cycling patterns in the unspliced-spliced space of GOterm cell_cycle
  genes and you need a continuous, circular cell cycle trajectory per cell.
when_to_use_negative:
- scRNA-seq data lacking spliced/unspliced quantification (no velocyto/scVelo moments),
  since DeepCycle requires imputed Mu and Ms layers
- Datasets where no cycling genes are detected after the hotelling filter — README
  warns the results will not be reliable and deeper sequencing may be needed
- Assigning cell cycle phases based solely on gene markers in heterogeneous models,
  which the paper notes lacks generalizability across cellular models
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Autoencoder that fits circular unspliced-spliced patterns of cell cycle genes
    and outputs the per-cell angle cell_cycle_theta
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: 'Preprocessing: scvelo.pp.moments computes the imputed Mu and Ms layers that
    serve as DeepCycle input; its dynamical model is the baseline whose fitting failure
    motivates this skill'
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Upstream preprocessing of the input Anndata (spliced/unspliced quantification)
- name: numpy
  role: Numerical dependency of the DeepCycle environment (tested on 1.19.1)
- name: scipy
  role: Numerical dependency of the DeepCycle environment (tested on 1.5.2)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:5271e5e456bcfa49ede7222f797f33a0cc8ca2185f0db1b2a7484e2a19e1f6c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# unspliced-spliced-dynamics-fitting

## Summary

Fit circular unspliced-spliced (Mu/Ms) dynamics of cell cycle genes in scRNA-seq data using DeepCycle, a deep-learning method based on RNA velocity, to assign each cell a continuous transcriptional phase (cell_cycle_theta). Motivated by scVelo's dynamical model failing to fit single-gene cycling patterns.

## When to use

Use when a velocyto/scVelo-processed scRNA-seq dataset (h5ad with Mu/Ms layers) shows cycling patterns in the unspliced-spliced space of GOterm cell_cycle genes and you need a continuous, circular cell cycle trajectory per cell.

## When NOT to use

- scRNA-seq data lacking spliced/unspliced quantification (no velocyto/scVelo moments), since DeepCycle requires imputed Mu and Ms layers
- Datasets where no cycling genes are detected after the hotelling filter — README warns the results will not be reliable and deeper sequencing may be needed
- Assigning cell cycle phases based solely on gene markers in heterogeneous models, which the paper notes lacks generalizability across cellular models

## Inputs

- Anndata h5ad preprocessed with velocyto and scvelo.pp.moments (Mu/Ms layers), e.g. adata_mESC.h5ad or human fibroblast h5ad from Zenodo doi 10.5281/zenodo.4719436
- Gene list of candidate cycling genes for the organism of interest (e.g. GOterm cell_cycle annotation file)
- Base (initial-condition) gene for the transcriptional phase guess
- Minimum average expression threshold for gene inclusion (unspliced/spliced)

## Outputs

- Output Anndata with adata.obs['cell_cycle_theta'] circular phase per cell
- ./fits/ folder with unspliced-spliced pattern plots for fit-quality inspection
- ./DeepCycle_model/ TensorFlow model and input_data.npz of training inputs
- ./training/ plots of MSE during training
- ./hotelling/ genes showing more than one expression state (with --hotelling)
- Phase-transition thetas (G1/S, S/G2, M/G1) via estimate_cell_cycle_transitions.py

## How to apply

Preprocess the annotated h5ad (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436) with scvelo.pp.moments to obtain imputed Mu and Ms layers. Restrict inference to a curated cycling gene list (e.g. GOterm cell_cycle annotation for mouse or human) and supply a base gene as the initial phase condition (Nusap1 for mESC, MELK for human fibroblast leiden cluster 0); first run with a random gene plus the --hotelling flag to detect cycling genes showing at least two expression states. Apply an unspliced/spliced expression threshold (suggested around 1.0, or 0.5 for the mESC example) depending on sequencing depth. DeepCycle trains a two-step autoencoder: first the encoder on phases estimated from the base gene (atan2 of z-scored spliced and unspliced reads), then encoder+decoder to reconstruct unspliced-spliced reads, using Adam on MSE with EarlyStopping (patience 20) and ReduceLROnPlateau (factor 0.8, patience 5), 17% validation cells and batch size 5. Rationale: scVelo's extended dynamical model could not fit the actual single-gene cycling dynamics nor recover correct latent time, so constant-rate assumptions are relaxed in favor of a flexible autoencoder that infers a circular latent angle theta.

## Related tools

- **DeepCycle** (Autoencoder that fits circular unspliced-spliced patterns of cell cycle genes and outputs the per-cell angle cell_cycle_theta) — https://github.com/andreariba/DeepCycle
- **scVelo** (Preprocessing: scvelo.pp.moments computes the imputed Mu and Ms layers that serve as DeepCycle input; its dynamical model is the baseline whose fitting failure motivates this skill) — https://scvelo.readthedocs.io/
- **velocyto** (Upstream preprocessing of the input Anndata (spliced/unspliced quantification))
- **numpy** (Numerical dependency of the DeepCycle environment (tested on 1.19.1))
- **scipy** (Numerical dependency of the DeepCycle environment (tested on 1.5.2))

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Inspect ./fits/ unspliced-spliced pattern plots per gene — the fits should trace the observed circular patterns rather than the failed constant-rate scVelo fits of Supplementary Fig. S3
- Confirm the output Anndata contains a finite, circular adata.obs['cell_cycle_theta'] for all cells
- Check ./training/ MSE curves show plateau-based convergence (EarlyStopping patience 20, ReduceLROnPlateau factor 0.8)
- Run estimate_cell_cycle_transitions.py to recover G1/S, S/G2 and M/G1 transition thetas consistent with the annotated cell cycle phases
- Verify the inferred theta agrees with DeepCycle's continuous trajectory in comparative panels (Fig. 2, Supplementary Figs. S3, S7, S9)

## Limitations

- If no cycling genes are found, results are unreliable and deeper sequencing (more RNAs per cell) may be required
- Wrong unspliced-spliced fits can occur even when cycling patterns are present
- The preprocessing step with scvelo.pp.moments is mandatory in the current version
- The method assigns a transcriptional angle, not phases directly; phase labels require downstream estimation of transition thetas (G1/S, S/G2, M/G1)

## Evidence

- [readme] Preprocessing requires scVelo moments: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [methods] Expression threshold around 1.0: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [supplementary] Encoder trained on atan2 phase estimate: "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [readme] Fits folder for quality control: "./fits/: the folder contains unspliced-spliced patterns to check if the fitting procedure worked properly"
- [readme] No cycling genes means unreliable results: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper"
