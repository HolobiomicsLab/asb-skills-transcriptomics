---
name: sc-rna-seq-subpopulation-mapping
description: Use when scRNA-seq data with spliced/unspliced layers contains a proliferative
  subpopulation (e.g. Leiden cluster 0 in human fibroblasts) and the goal is to resolve
  the G1-to-G0 exit path and its marker genes.
when_to_use_negative:
- If the dataset contains no cycling genes after filtering — the results will not
  be reliable and deeper sequencing may be needed.
- If the input Anndata lacks spliced/unspliced layers or has not been preprocessed
  with scvelo.pp.moments — DeepCycle requires imputed Mu/Ms moments.
- If the biological question does not concern cell-cycle dynamics or phase assignment,
  e.g. static differential expression between unrelated cell types.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: DeepCycle
  role: Deep-learning autoencoder that assigns a continuous cell-cycle angle (cell_cycle_theta)
    to each cell from RNA velocity unspliced-spliced patterns
  repo: github.com/andreariba/DeepCycle
- name: scVelo
  role: Preprocessing step pp.moments to compute imputed unspliced (Mu) and spliced
    (Ms) layers required by DeepCycle
- name: velocyto
  role: Upstream spliced/unspliced count quantification feeding the input Anndata
- name: TensorFlow
  role: Backend training the DeepCycle encoder-decoder with Adam/MSE, EarlyStopping
    and ReduceLROnPlateau callbacks
- name: Python
  role: Execution environment (tested on 3.7.9) for DeepCycle.py and scvelo preprocessing
- name: numpy
  role: Numerical dependency of the DeepCycle environment (tested on 1.19.1)
- name: scipy
  role: Scientific dependency of the DeepCycle environment (tested on 1.5.2)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:6130377af5004be71ddf2b118cf05c4ddca2a13d6574bb4e000cd690176a51c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCycle
  - Python
  - TensorFlow
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sc-rna-seq-subpopulation-mapping

## Summary

Map a single-cell RNA-seq subpopulation (Leiden cluster) along a continuous cell-cycle trajectory with DeepCycle to identify genes and transcription factors underlying the branching point where human fibroblasts exit the cell cycle into a nonproliferative (G0-like) state.

## When to use

Use when scRNA-seq data with spliced/unspliced layers contains a proliferative subpopulation (e.g. Leiden cluster 0 in human fibroblasts) and the goal is to resolve the G1-to-G0 exit path and its marker genes.

## When NOT to use

- If the dataset contains no cycling genes after filtering — the results will not be reliable and deeper sequencing may be needed.
- If the input Anndata lacks spliced/unspliced layers or has not been preprocessed with scvelo.pp.moments — DeepCycle requires imputed Mu/Ms moments.
- If the biological question does not concern cell-cycle dynamics or phase assignment, e.g. static differential expression between unrelated cell types.

## Inputs

- Anndata (.h5ad) preprocessed with velocyto and scvelo.pp.moments (Ms/Mu layers)
- Organism-specific cell-cycle gene list (e.g. GOterm cell_cycle annotation text file)
- Base gene name used as initial phase condition (e.g. MELK for human fibroblasts)
- Expression threshold value (suggested around 1.0, sequencing-depth dependent)

## Outputs

- Annotated Anndata with adata.obs['cell_cycle_theta'] per cell (e.g. fibroblast_deepcycle_annotated.h5ad)
- ./fits/ unspliced-spliced pattern plots for fit verification
- ./training/ MSE training plots
- ./DeepCycle_model/ TensorFlow model and input_data.npz
- ./hotelling/ list of genes with more than one expression state (with --hotelling)
- Theta estimates for phase transitions (G1/S, S/G2, M/G1) from estimate_cell_cycle_transitions.py

## How to apply

Subset the subpopulation of interest (for human fibroblasts, the Leiden cluster 0) from the Anndata object preprocessed with scvelo.pp.moments so that imputed unspliced (Mu) and spliced (Ms) layers exist. Run DeepCycle.py with the organism-specific cell-cycle gene list (e.g. GOterm cell_cycle human annotation), a cycling base gene chosen as initial condition for the transcriptional phase (MELK for human fibroblasts, Nusap1 for mESCs), an expression_threshold around 1.0 (adjusted to sequencing depth; mESC example used 0.5), and optionally the --hotelling filter to flag genes showing at least two expression states and --gpu. Inspect ./fits/ unspliced-spliced pattern plots and ./training/ MSE plots to confirm the autoencoder fitting worked; absence of cycling genes indicates unreliable results. Extract adata.obs['cell_cycle_theta'] and use it, optionally with estimate_cell_cycle_transitions.py for G1/S, S/G2 and M/G1 transition thetas, to order cells along the G1-to-G0 path and compare differential markers (MXD4, CDC42EP5, CLIP3, MIR22HG upregulated; ESCO2, MCM10, MYBL2, NUF2 and FOXM1 downregulated) and motifs such as E2F7_E2F1.

## Related tools

- **DeepCycle** (Deep-learning autoencoder that assigns a continuous cell-cycle angle (cell_cycle_theta) to each cell from RNA velocity unspliced-spliced patterns) — github.com/andreariba/DeepCycle
- **scVelo** (Preprocessing step pp.moments to compute imputed unspliced (Mu) and spliced (Ms) layers required by DeepCycle)
- **velocyto** (Upstream spliced/unspliced count quantification feeding the input Anndata)
- **TensorFlow** (Backend training the DeepCycle encoder-decoder with Adam/MSE, EarlyStopping and ReduceLROnPlateau callbacks)
- **Python** (Execution environment (tested on 3.7.9) for DeepCycle.py and scvelo preprocessing)
- **numpy** (Numerical dependency of the DeepCycle environment (tested on 1.19.1))
- **scipy** (Scientific dependency of the DeepCycle environment (tested on 1.5.2))

## Examples

```
python DeepCycle.py --input_adata fibroblast_cluster0.h5ad --gene_list GO_cell_cycle_annotation_human.txt --base_gene MELK --expression_threshold 1.0 --gpu --hotelling --output_adata fibroblast_deepcycle_annotated.h5ad
```

## Evaluation signals

- ./fits/ unspliced-spliced pattern plots show plausible fits of the cycling gene trajectories; if the fits are wrong the results are unreliable.
- ./training/ MSE plots show both training steps converged with EarlyStopping/ReduceLROnPlateau (patience 20 and 5 respectively).
- adata.obs['cell_cycle_theta'] is a circular variable whose ordering reproduces expected phase-marker dynamics (FOXM1 downregulation toward G0; MXD4, CDC42EP5, CLIP3, MIR22HG upregulated along the mid-G1-to-G0 path).
- At least some cycling genes are detected — 'If you do not find any cycling genes the results will not be reliable.'
- estimate_cell_cycle_transitions.py successfully estimates distinct thetas for the G1/S, S/G2 and M/G1 transitions from cell_cycle_theta.

## Limitations

- Defining cell cycle phases in RNA data based solely on gene markers lacks generalizability across cellular models; future methods should rely on dynamical features of gene expression.
- If no cycling genes are found, results are unreliable and deeper sequencing (more RNAs per cell) may be required.
- An extensive study of transcriptional changes during the cell cycle as cells reach confluence is still missing.
- Extension of DeepCycle as a Variational Autoencoder to model distributions of unspliced-spliced RNAs is envisioned but not yet implemented.

## Evidence

- [supplementary] Subpopulation subsetting rationale: "To reproduce the results for the human fibroblast, you have to run the DeepCycle on the subpopulation identified by the leiden clustering as 0"
- [readme] Base gene and hotelling strategy: "a gene to use as initial condition for the transcriptional phase. A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the"
- [readme] Output variable: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes"
- [methods] Preprocessing requirement: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [methods] Reliability warning: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
- [supplementary] Transition detection script: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [results] G0 exit marker genes: [evidence span withheld — anchor status: not_found]
