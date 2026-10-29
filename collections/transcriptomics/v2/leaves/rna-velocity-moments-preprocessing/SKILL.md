---
name: rna-velocity-moments-preprocessing
description: Use when preparing an scRNA-seq AnnData (e.g. adata_mESC.h5ad from Zenodo)
  that lacks imputed 'Ms'/'Mu' layers, as a mandatory step before running DeepCycle
  or any unspliced-spliced RNA velocity analysis.
when_to_use_negative:
- The AnnData already contains imputed 'Ms' and 'Mu' layers (re-running moments would
  be redundant)
- The dataset has no velocyto-derived spliced/unspliced count layers, so moments imputation
  cannot be computed
- The downstream analysis does not rely on unspliced-spliced RNA velocity dynamics
  (e.g. standard expression clustering or marker analysis)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scVelo
  role: provides pp.moments, which computes the neighbor graph and generates the imputed
    'Ms' and 'Mu' layers required by DeepCycle
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: upstream quantification of spliced and unspliced reads that produces the AnnData
    layers on which moments imputation operates
- name: DeepCycle
  role: downstream consumer of the preprocessed AnnData (input_adata must be preprocessed
    with velocyto, scvelo moments) for cell-cycle phase inference
  repo: https://github.com/andreariba/DeepCycle
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:bca94e7d1a0b145588155e61addb3f783e6cc276b8140825d613bde668f1583e
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# rna-velocity-moments-preprocessing

## Summary

Preprocessing skill that runs scvelo's pp.moments on a velocyto-annotated AnnData to generate the imputed spliced ('Ms') and unspliced ('Mu') expression layers that DeepCycle requires as input for cell-cycle transcriptional phase inference.

## When to use

Use when preparing an scRNA-seq AnnData (e.g. adata_mESC.h5ad from Zenodo) that lacks imputed 'Ms'/'Mu' layers, as a mandatory step before running DeepCycle or any unspliced-spliced RNA velocity analysis.

## When NOT to use

- The AnnData already contains imputed 'Ms' and 'Mu' layers (re-running moments would be redundant)
- The dataset has no velocyto-derived spliced/unspliced count layers, so moments imputation cannot be computed
- The downstream analysis does not rely on unspliced-spliced RNA velocity dynamics (e.g. standard expression clustering or marker analysis)

## Inputs

- AnnData .h5ad file with spliced/unspliced count layers preprocessed with velocyto (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- scVelo neighbor-graph parameters implicit in pp.moments (n_pcs/n_neighbors)

## Outputs

- AnnData with imputed spliced layer adata.layers['Ms'] and imputed unspliced layer adata.layers['Mu']
- Preprocessed .h5ad suitable as --input_adata for python DeepCycle.py

## How to apply

Start from an AnnData (.h5ad) whose spliced and unspliced counts have been quantified with velocyto. Run scvelo's pp.moments, which first computes the neighbor graph and then smooths the spliced/unspliced read counts, producing the imputed layers 'Ms' and 'Mu'. The paper's supplementary specifies that the DeepCycle input file must be 'preprocessed with velocyto, scvelo (moments)'. This is a required preprocessing step in the current DeepCycle version: the imputed unspliced (Mu) and spliced (Ms) expressions are what the autoencoder consumes (per-cell phases are seeded from the atan2 of z-scored spliced and unspliced reads of a base gene such as Nusap1 for mESCs or MELK for human fibroblasts). Verify the output AnnData actually carries non-integer, smoothed 'Ms' and 'Mu' layers before handing it to DeepCycle.py.

## Related tools

- **scVelo** (provides pp.moments, which computes the neighbor graph and generates the imputed 'Ms' and 'Mu' layers required by DeepCycle) — https://scvelo.readthedocs.io/
- **velocyto** (upstream quantification of spliced and unspliced reads that produces the AnnData layers on which moments imputation operates)
- **DeepCycle** (downstream consumer of the preprocessed AnnData (input_adata must be preprocessed with velocyto, scvelo moments) for cell-cycle phase inference) — https://github.com/andreariba/DeepCycle

## Examples

```
import scvelo as scv; adata = scv.read('adata_mESC.h5ad'); scv.pp.moments(adata)
```

## Evaluation signals

- The output AnnData contains both adata.layers['Ms'] and adata.layers['Mu'] with smoothed (non-integer) imputed values
- The resulting .h5ad is accepted as --input_adata by DeepCycle.py (the tool expects an input 'preprocessed with velocyto and scvelo (moments)')
- DeepCycle's first training stage can compute the atan2 of z-scored spliced and unspliced reads of the base gene, confirming the imputed layers are populated
- The preprocessed file matches the annotation used by estimate_cell_cycle_transitions.py, which requires obs['cell_cycle_theta'] plus the 'Ms' and 'Mu' layers from scVelo

## Limitations

- DeepCycle's results are unreliable if no cycling genes are found after preprocessing; the authors note it may be necessary to sequence deeper and increase the number of RNAs per cell
- The imputation quality depends on the velocyto spliced/unspliced annotation and neighbor-graph parameters; wrong or missing unspliced counts cannot be repaired by pp.moments
- In the current DeepCycle version this preprocessing step is mandatory — DeepCycle cannot run directly on raw counts

## Evidence

- [methods] In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [supplementary] Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle.: "Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle."
- [readme] to generate imputed unspliced (Mu) and spliced (Ms) expressions.: "to generate imputed unspliced (Mu) and spliced (Ms) expressions."
- [supplementary] the layers 'Ms' and 'Mu' from scVelo.: "the layers 'Ms' and 'Mu' from scVelo."
