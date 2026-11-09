---
name: single-cell-unspliced-rna-layer-processing
description: Use when an scRNA-seq anndata contains spliced/unspliced count layers
  and downstream analysis (DeepCycle cell-cycle phase assignment, motif activity from
  unspliced counts) requires scVelo-computed moments.
when_to_use_negative:
- No cycling genes can be detected after filtering — the article warns results will
  not be reliable, likely because sequencing depth (RNAs per cell) is insufficient
- The anndata lacks velocyto-derived spliced/unspliced layers (e.g. only a gene expression
  matrix is available), since pp.moments and DeepCycle require unspliced counts
- The goal is cell-cycle phase assignment from gene markers alone rather than dynamical
  unspliced-spliced features — the paper notes marker-based definitions lack generalizability
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scVelo
  role: computes the imputed moment layers 'Ms' and 'Mu' via pp.moments, required
    as DeepCycle input
- name: velocyto
  role: produces the spliced/unspliced count annotations in the anndata prior to moment
    computation
- name: DeepCycle
  role: consumes the moment-preprocessed anndata to infer the cell_cycle_theta circular
    trajectory
  repo: https://github.com/andreariba/DeepCycle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:c9953808ab8df22e443db2ca07c8d90365e89ca12b4ae0cc3eeeb9f082f00c66
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
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-unspliced-rna-layer-processing

## Summary

Prepare scRNA-seq anndata objects by computing scVelo moment layers (imputed unspliced 'Mu' and spliced 'Ms') so that unspliced-spliced cycling patterns can be fed to downstream methods such as DeepCycle or ISMARA-style motif activity models.

## When to use

Use when an scRNA-seq anndata contains spliced/unspliced count layers and downstream analysis (DeepCycle cell-cycle phase assignment, motif activity from unspliced counts) requires scVelo-computed moments.

## When NOT to use

- No cycling genes can be detected after filtering — the article warns results will not be reliable, likely because sequencing depth (RNAs per cell) is insufficient
- The anndata lacks velocyto-derived spliced/unspliced layers (e.g. only a gene expression matrix is available), since pp.moments and DeepCycle require unspliced counts
- The goal is cell-cycle phase assignment from gene markers alone rather than dynamical unspliced-spliced features — the paper notes marker-based definitions lack generalizability

## Inputs

- anndata (.h5ad) scRNA-seq file preprocessed with velocyto containing spliced and unspliced count layers
- gene list of candidate cycling genes for the organism (e.g. GOterm cell_cycle annotation file)
- base (seed) gene name used as initial condition for the transcriptional phase
- minimum average expression threshold (suggested ~1.0, data-dependent)

## Outputs

- anndata with scVelo moment layers 'Ms' (imputed spliced) and 'Mu' (imputed unspliced)
- anndata with adata.obs['cell_cycle_theta'] circular cell-cycle phase variable after DeepCycle
- supporting run artifacts: ./fits/ unspliced-spliced pattern plots, ./DeepCycle_model/ TensorFlow model, ./training/ MSE plots, input_data.npz

## How to apply

Start from an anndata file that has been preprocessed with velocyto to obtain spliced and unspliced counts, then run scVelo's pp.moments to generate the imputed unspliced ('Mu') and spliced ('Ms') expression layers, which DeepCycle requires as input. Restrict inference to a candidate cycling gene list for the organism (e.g. GOterm cell_cycle) and apply a minimum average expression threshold around 1.0 (e.g. 0.5 was used for the mESC dataset on Zenodo), tuning it to sequencing depth per cell. Optionally enable the Hotelling filter to retain genes showing at least two distinct expression states, and choose a base gene (e.g. Nusap1 for mESC, MELK for human fibroblast leiden cluster 0) to initialize the transcriptional phase. The resulting anndata carries obs['cell_cycle_theta'] plus the 'Ms' and 'Mu' layers, which are also the required inputs for phase-transition estimation scripts and for promoter-level motif activity regression on unspliced reads.

## Related tools

- **scVelo** (computes the imputed moment layers 'Ms' and 'Mu' via pp.moments, required as DeepCycle input)
- **velocyto** (produces the spliced/unspliced count annotations in the anndata prior to moment computation)
- **DeepCycle** (consumes the moment-preprocessed anndata to infer the cell_cycle_theta circular trajectory) — https://github.com/andreariba/DeepCycle

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Output anndata contains both scVelo layers 'Ms' and 'Mu' and obs['cell_cycle_theta'] as required by estimate_cell_cycle_transitions.py
- Cycling patterns are visible in the unspliced-spliced fits under ./fits/, indicating the moment preprocessing supported proper phase inference
- Genes pass the expression threshold (~1.0 average, 0.5 used for mESC) and, with --hotelling, selected genes show more than one state of expression in ./hotelling/
- Downstream validation reproduces published results, e.g. the Ybx1 motif-mRNA correlation r=0.91 (p=4e-19) in ductal cell progenitors within tolerance

## Limitations

- If no cycling genes are found the results are unreliable; deeper sequencing may be needed to increase RNAs per cell
- Suggested expression threshold (~1.0) depends on sequencing depth per cell and must be adjusted per dataset
- The article notes previous constant-rate RNA velocity assumptions may need relaxing, and marker-based phase definitions lack generalizability across cellular models

## Evidence

- [readme] In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [supplementary] Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle.: "Anndata input file preprocessed with velocyto and scvelo (moments)."
- [readme] If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
- [supplementary] the layers 'Ms' and 'Mu' from scVelo: "The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo."
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
