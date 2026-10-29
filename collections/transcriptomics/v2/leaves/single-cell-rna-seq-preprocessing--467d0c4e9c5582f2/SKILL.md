---
name: single-cell-rna-seq-preprocessing
description: Use when an .h5ad Anndata from velocyto lacks imputed spliced/unspliced
  moments and must be prepared before running DeepCycle or scVelo-based cell cycle
  fitting.
when_to_use_negative:
- The Anndata already contains 'Ms' and 'Mu' moment layers from scvelo (recomputing
  is unnecessary)
- Only raw UMI count matrices without velocyto-derived spliced/unspliced annotation
  are available and no loom/velocyto quantification can be obtained
- The analysis goal does not involve RNA velocity or DeepCycle (e.g. purely transcriptomic
  clustering), where moment imputation adds no value
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scVelo
  role: Provides scvelo.pp.moments, the required preprocessing step that computes
    imputed 'Ms' and 'Mu' layers from spliced/unspliced counts
  repo: https://github.com/andreariba/DeepCycle
- name: velocyto
  role: Generates the spliced/unspliced Anndata that is preprocessed upstream of the
    moments computation
  repo: https://github.com/andreariba/DeepCycle
- name: DeepCycle
  role: Downstream consumer of the preprocessed Anndata (requires 'Ms'/'Mu' layers)
    for cell cycle theta inference
  repo: https://github.com/andreariba/DeepCycle
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:902c4017cc6ab168db6c79be27c97337a00897f74444831fe775af62bda1cbde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Docker
  - GRouNdGAN
  - PyTorch
  - Python
  - SPARSIM
  - TensorBoard
  - TensorFlow
  - numpy
  - scDESIGN2
  - scGAN
  - scVelo
  - scanpy
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# single-cell-rna-seq-preprocessing

## Summary

Preprocess scRNA-seq spliced/unspliced count matrices with scvelo.pp.moments (on Anndata preprocessed with velocyto) to produce imputed Mu and Ms layers, which are the mandatory input for DeepCycle's cell cycle inference.

## When to use

Use when an .h5ad Anndata from velocyto lacks imputed spliced/unspliced moments and must be prepared before running DeepCycle or scVelo-based cell cycle fitting.

## When NOT to use

- The Anndata already contains 'Ms' and 'Mu' moment layers from scvelo (recomputing is unnecessary)
- Only raw UMI count matrices without velocyto-derived spliced/unspliced annotation are available and no loom/velocyto quantification can be obtained
- The analysis goal does not involve RNA velocity or DeepCycle (e.g. purely transcriptomic clustering), where moment imputation adds no value

## Inputs

- Anndata (.h5ad) file preprocessed with velocyto and containing spliced/unspliced layers (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- Cell-cycle gene list for the organism (e.g. GOterm cell_cycle annotation file such as GO_cell_cycle_annotation_mouse.txt)
- Expression threshold value (suggested around 1.0, adjusted to sequencing depth per cell)

## Outputs

- Anndata with imputed moments layers 'Ms' (spliced) and 'Mu' (unspliced) computed by scvelo.pp.moments, ready as --input_adata for DeepCycle.py
- Optionally subset Anndata (e.g. fibroblast leiden cluster 0) with the organism-specific cell cycle gene annotation attached

## How to apply

Start from a velocyto-annotated Anndata containing spliced and unspliced count layers and run scvelo's preprocessing: scvelo.pp.moments to generate the imputed 'Ms' and 'Mu' layers. Subset the data when needed, e.g. for the human fibroblast dataset restrict to the subpopulation identified by leiden clustering as 0. Select the candidate cycling gene set for the organism of interest, e.g. the GOterm cell_cycle list (go_annotation/GO_cell_cycle_annotation_mouse.txt for mESC), and pass an expression threshold (suggested around 1.0, adjusted to sequencing depth per cell; 0.5 was used for the mESC dataset) so only sufficiently expressed genes are retained. The resulting Anndata with obs, 'Ms'/'Mu' layers and gene list is the direct input to DeepCycle.py; without this moments preprocessing step DeepCycle cannot run.

## Related tools

- **scVelo** (Provides scvelo.pp.moments, the required preprocessing step that computes imputed 'Ms' and 'Mu' layers from spliced/unspliced counts) — https://github.com/andreariba/DeepCycle
- **velocyto** (Generates the spliced/unspliced Anndata that is preprocessed upstream of the moments computation) — https://github.com/andreariba/DeepCycle
- **DeepCycle** (Downstream consumer of the preprocessed Anndata (requires 'Ms'/'Mu' layers) for cell cycle theta inference) — https://github.com/andreariba/DeepCycle

## Examples

```
import scvelo as scv; adata = scv.read('adata_mESC.h5ad'); scv.pp.moments(adata)
```

## Evaluation signals

- After scvelo.pp.moments, the Anndata contains non-empty layers 'Ms' and 'Mu' with the same shape as the gene-by-cell matrix
- The input file satisfies DeepCycle's stated requirement: 'Anndata input file preprocessed with velocyto and scvelo (moments)'
- Genes below the chosen expression threshold (~1.0 average expression, or 0.5 for the mESC run) are excluded so the retained set yields detectable cycling patterns
- Running DeepCycle on the preprocessed data finds cycling genes (via the --hotelling filter) rather than reporting no cycling genes, which would indicate unreliable results
- For the fibroblast dataset, preprocessing is applied to the leiden cluster 0 subpopulation only, matching the published analysis scope

## Limitations

- If no cycling genes are found after preprocessing, the results will not be reliable; deeper sequencing (more RNAs per cell) may be required
- Expression threshold must be tuned to sequencing depth per cell; a fixed value is not universally appropriate
- The gene-marker-based definition of cell cycle phases lacks generalizability across cellular models due to variability in cell cycle signatures

## Evidence

- [methods] In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [readme] Anndata input file preprocessed with velocyto and scvelo (moments).: "Anndata input file preprocessed with velocyto and scvelo (moments)."
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [readme] To reproduce the results for the human fibroblast, you have to run the DeepCycle on the subpopulation identified by the leiden clustering as 0: "To reproduce the results for the human fibroblast, you have to run the DeepCycle on the subpopulation identified by the leiden clustering as 0"
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable."
