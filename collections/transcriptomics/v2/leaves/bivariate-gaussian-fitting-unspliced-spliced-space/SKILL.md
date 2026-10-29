---
name: bivariate-gaussian-fitting-unspliced-spliced-space
description: 'Use when selecting reliable cycling-gene inputs for DeepCycle: candidate
  genes from a GOterm cell_cycle list must be checked in unspliced-spliced space (scVelo
  Ms/Mu layers) for multiple maxima before phase inference.'
when_to_use_negative:
- Genes that show only a single expression state in unspliced-spliced space — the
  Hotelling filter will correctly exclude them, so do not force them into the reliable-gene
  list
- Data lacking scVelo-computed 'Ms' and 'Mu' moment layers — the mixture fit requires
  the imputed unspliced-spliced reads from pp.moments
- Datasets where no genes pass the expression threshold and zero cycling genes are
  found — the paper warns such DeepCycle results will not be reliable and sequencing
  depth should be increased instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: DeepCycle
  role: Implements the --hotelling filter that fits the bivariate Gaussian mixtures
    and writes the selected multi-maxima genes to ./hotelling/
  repo: github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides pp.moments preprocessing producing the imputed Ms and Mu layers fed
    to the mixture fitting
  repo: https://scvelo.readthedocs.io/
- name: scipy
  role: Used (with numpy) to fit the two-component bivariate Gaussian mixture to the
    (spliced, unspliced) read distribution
- name: numpy
  role: Numerical backend for the Gaussian mixture fitting of unspliced-spliced reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:355a163952c64bb23e768494179f5fc21cfcacf3e12bc072fa596cdbe33d3b0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCycle
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bivariate-gaussian-fitting-unspliced-spliced-space

## Summary

Identifies reliably cycling genes for DeepCycle by fitting a two-component bivariate Gaussian mixture to each gene's (spliced, unspliced) read distribution and applying a Hotelling T2 test, retaining genes whose mixture shows at least two distinct expression maxima. This isolates genes with more than one expression state, which underpin robust transcriptional-phase inference (n = 158 such genes for the mESC analysis).

## When to use

Use when selecting reliable cycling-gene inputs for DeepCycle: candidate genes from a GOterm cell_cycle list must be checked in unspliced-spliced space (scVelo Ms/Mu layers) for multiple maxima before phase inference.

## When NOT to use

- Genes that show only a single expression state in unspliced-spliced space — the Hotelling filter will correctly exclude them, so do not force them into the reliable-gene list
- Data lacking scVelo-computed 'Ms' and 'Mu' moment layers — the mixture fit requires the imputed unspliced-spliced reads from pp.moments
- Datasets where no genes pass the expression threshold and zero cycling genes are found — the paper warns such DeepCycle results will not be reliable and sequencing depth should be increased instead

## Inputs

- AnnData (.h5ad) preprocessed with velocyto and scVelo pp.moments, containing imputed 'Ms' and 'Mu' layers
- Organism-specific candidate cycling gene list (e.g. GOterm cell_cycle annotation file such as go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Average-expression threshold value (suggested ~1.0, depending on sequencing depth per cell)

## Outputs

- List of cycling genes with at least two distinct expression states (multi-maxima in unspliced-spliced space), written to the ./hotelling/ subdirectory
- Filtered gene set usable as reliable input for DeepCycle phase inference (n = 158 genes for the mESC dataset, recorded in Supplementary Data 1)

## How to apply

Load the AnnData preprocessed with velocyto and scVelo pp.moments (imputed Ms and Mu layers), and restrict the analysis to candidate cycling genes from the organism-specific list (e.g. GOterm cell_cycle) that pass the average-expression threshold (~1.0 suggested; 0.5 was used in the mESC run). For each gene, fit a two-component bivariate Gaussian mixture to the (spliced, unspliced) read distribution using scipy/numpy. Apply the Hotelling T2 significance test to assess whether the two fitted Gaussian components are statistically distinguishable. Retain only genes whose fitted mixture exhibits at least two distinct maxima — i.e. more than one expression state — and write the resulting gene list to the ./hotelling/ output directory. The rationale is that genes cycling through multiple states in unspliced-spliced space provide reliable phase estimates; genes with a single state are filtered out.

## Related tools

- **DeepCycle** (Implements the --hotelling filter that fits the bivariate Gaussian mixtures and writes the selected multi-maxima genes to ./hotelling/) — github.com/andreariba/DeepCycle
- **scVelo** (Provides pp.moments preprocessing producing the imputed Ms and Mu layers fed to the mixture fitting) — https://scvelo.readthedocs.io/
- **scipy** (Used (with numpy) to fit the two-component bivariate Gaussian mixture to the (spliced, unspliced) read distribution)
- **numpy** (Numerical backend for the Gaussian mixture fitting of unspliced-spliced reads)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Genes retained in ./hotelling/ each show more than one state of expression (at least two distinct maxima in the fitted bivariate Gaussian mixture)
- The two fitted Gaussian components are statistically distinguishable by the Hotelling T2 test
- Reconstructed gene set agrees in membership and count with the n = 158 cycling genes reported in Supplementary Data 1 of the paper
- All retained genes pass the average-expression threshold (suggested ~1.0; 0.5 for the mESC run)
- If the output list is empty, results are unreliable — no cycling genes found indicates the data lack sufficient RNAs per cell

## Limitations

- The Hotelling filter is optional in DeepCycle; skipping it can let single-state genes through, weakening phase robustness
- If no cycling genes are found, the results will not be reliable — the authors suggest sequencing deeper to increase the number of RNAs per cell
- The suggested average-expression threshold (~1.0) depends on sequencing depth per cell and must be adjusted per dataset (0.5 for the mESC Zenodo dataset)

## Evidence

- [readme] The subdirectory DeepCycle/hotelling will show genes selected to show at least two different states of expression.: "The subdirectory DeepCycle/hotelling will show genes selected to show at least two different states of expression."
- [methods] list of possible cycling genes for the organism of interest (e.g. GOterm cell_cycle): "list of possible cycling genes for the organism of interest (e.g. GOterm cell_cycle)"
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [supplementary] Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle.: "Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle."
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
