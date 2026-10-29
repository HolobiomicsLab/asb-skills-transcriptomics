---
name: umi-count-aggregation-per-phase
description: Use when phase-annotated h5ad data (adata.obs['cell_cycle_theta'] from
  a DeepCycle run) is available and you need to verify that UMI counts per cell increase
  roughly 2-fold along the cell cycle prior to the mitotic drop.
when_to_use_negative:
- The h5ad lacks a DeepCycle cell_cycle_theta annotation — the circular binning requires
  a continuous transcriptional phase; rerun DeepCycle first (e.g., with pp.moments
  from scVelo preprocessing).
- No cycling genes were found in the dataset — DeepCycle warns the results will not
  be reliable in this case, so UMI-vs-phase aggregation would be meaningless.
- The question is about gene-level cycling expression dynamics rather than bulk RNA
  content per cell; use the DeepCycle gene fitting workflow instead.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: DeepCycle
  role: Produces the adata.obs['cell_cycle_theta'] transcriptional phase used as the
    circular binning variable for UMI aggregation
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the pp.moments preprocessing (Ms/Mu layers) required before DeepCycle
    phase annotation of the h5ad inputs
  repo: https://github.com/theislab/scvelo
- name: numpy
  role: Circular binning of cells by cell_cycle_theta and averaging UMI counts per
    bin
- name: scipy
  role: Curve fitting of the exponential doubling trend to the binned UMI counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:30dde7115495ff6662baa3bbc38331f0547bc5361c9c1ab461afa23df30cd024
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# umi-count-aggregation-per-phase

## Summary

Quantifies total RNA content (UMI counts) per cell binned circularly by DeepCycle's cell_cycle_theta to test whether RNAs per cell approximately double (~2.0–2.1 fold change) across the transcriptional phase before mitosis.

## When to use

Use when phase-annotated h5ad data (adata.obs['cell_cycle_theta'] from a DeepCycle run) is available and you need to verify that UMI counts per cell increase roughly 2-fold along the cell cycle prior to the mitotic drop.

## When NOT to use

- The h5ad lacks a DeepCycle cell_cycle_theta annotation — the circular binning requires a continuous transcriptional phase; rerun DeepCycle first (e.g., with pp.moments from scVelo preprocessing).
- No cycling genes were found in the dataset — DeepCycle warns the results will not be reliable in this case, so UMI-vs-phase aggregation would be meaningless.
- The question is about gene-level cycling expression dynamics rather than bulk RNA content per cell; use the DeepCycle gene fitting workflow instead.

## Inputs

- Phase-annotated h5ad files (adata_mESC.h5ad, human fibroblast h5ad) with obs['cell_cycle_theta'] from Zenodo 10.5281/zenodo.4719436
- Public ductal cell progenitor scRNA-seq dataset annotated with cell_cycle_theta from a DeepCycle run
- Total UMI counts per cell (e.g., adata.obs n_counts) summed from the count matrix

## Outputs

- UMI-vs-phase profile (average UMI counts per circular phase bin) per dataset
- Fitted exponential doubling trend over the pre-mitotic phase interval
- umi_fold_changes.csv with fold changes per dataset (expected 2.1 mESCs, 2.0 ductal, 2.1 fibroblasts)

## How to apply

Download the phase-annotated h5ad files for mESCs and human fibroblasts from Zenodo (10.5281/zenodo.4719436) and the public ductal cell progenitor dataset, each carrying adata.obs['cell_cycle_theta'] from DeepCycle. For each dataset, bin cells circularly by cell_cycle_theta into phase bins spanning the full cycle, then sum total UMI counts per cell and average within each bin to get the UMI-vs-phase profile. Fit an exponential doubling trend over the phase interval preceding the mitotic drop using numpy/scipy curve fitting, and compute the fold change of the peak preceding mitosis relative to the preceding minimum. The doubling reflects the mother cell approximately doubling in volume before mitosis, so the fold change should be near 2.

## Related tools

- **DeepCycle** (Produces the adata.obs['cell_cycle_theta'] transcriptional phase used as the circular binning variable for UMI aggregation) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the pp.moments preprocessing (Ms/Mu layers) required before DeepCycle phase annotation of the h5ad inputs) — https://github.com/theislab/scvelo
- **numpy** (Circular binning of cells by cell_cycle_theta and averaging UMI counts per bin)
- **scipy** (Curve fitting of the exponential doubling trend to the binned UMI counts)

## Examples

```
import scanpy as sc; import numpy as np; adata = sc.read_h5ad('adata_mESC_DeepCycle.h5ad'); bins = np.floor(adata.obs['cell_cycle_theta']/(2*np.pi)*36); prof = adata.obs.groupby(bins)['n_counts'].mean()
```

## Evaluation signals

- Computed fold change reproduces 2.1 for mESCs, 2.0 for ductal cells, and 2.1 for fibroblasts within a small tolerance
- UMI-vs-phase profile shows a smooth positive trend peaking just before the mitotic drop in each dataset
- Fitted exponential trend over the pre-mitotic interval has a doubling consistent with the raw peak-to-trough ratio
- Bins span the full circular cycle (theta 0–2π) with cells present in every phase bin, so the profile is continuous and closes on itself

## Limitations

- If DeepCycle finds no cycling genes the results will not be reliable; deeper sequencing (more RNAs per cell) may be needed.
- Cell cycle signatures vary among cellular models, so the phase annotation and hence the UMI fold-change estimate may not generalize without a per-dataset DeepCycle run.
- Wrong fits of the unspliced-spliced patterns (inspectable in ./fits/) would corrupt the theta values underlying the aggregation.

## Evidence

- [results] RNA counts per cell as a function of the transcriptional phase show a positive fold change of 2.1 for mESCs, 2.0 for ductal cells, and 2.1 for fibroblasts: [evidence span withheld — anchor status: not_found]
- [supplementary] h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
