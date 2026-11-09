---
name: exponential-trend-fitting
description: Use when h5ad cells carry a continuous cell-cycle angle (cell_cycle_theta)
  and you must verify whether total UMI counts per cell double (fold change ~2.0-2.1)
  across the transcriptional phase before mitosis.
when_to_use_negative:
- The anndata lacks obs['cell_cycle_theta'] (DeepCycle was never run); fit the phase
  first rather than substituting gene-marker-based phases, which the paper notes lack
  generalizability
- No cycling genes were detected (per the README, 'If you do not find any cycling
  genes the results will not be reliable') — sequencing depth must first be increased
- The question concerns per-gene expression waves (e.g. G1 transcription waves), not
  global per-cell RNA content doubling
tools:
- name: DeepCycle
  role: Produced the circular cell_cycle_theta annotation in the h5ad files that defines
    the phase axis for binning cells
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the pp.moments preprocessing (Ms/Mu layers) required upstream of
    DeepCycle; preprocessed anndata is the DeepCycle input
  repo: https://scvelo.readthedocs.io/
- name: numpy
  role: Circular binning of cells by cell_cycle_theta and computation of per-bin mean
    UMI counts
- name: scipy
  role: Curve fitting of the exponential doubling trend to the binned UMI counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:e4a2ec0956d110dcffdbbc822d8dc82fcb25d1c405449c05b251110c8163b1f9
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# exponential-trend-fitting

## Summary

Quantify pre-mitotic doubling of cellular RNA content by fitting an exponential trend to binned UMI counts along the DeepCycle transcriptional phase (adata.obs['cell_cycle_theta']) and computing the fold change across the doubling segment.

## When to use

Use when h5ad cells carry a continuous cell-cycle angle (cell_cycle_theta) and you must verify whether total UMI counts per cell double (fold change ~2.0-2.1) across the transcriptional phase before mitosis.

## When NOT to use

- The anndata lacks obs['cell_cycle_theta'] (DeepCycle was never run); fit the phase first rather than substituting gene-marker-based phases, which the paper notes lack generalizability
- No cycling genes were detected (per the README, 'If you do not find any cycling genes the results will not be reliable') — sequencing depth must first be increased
- The question concerns per-gene expression waves (e.g. G1 transcription waves), not global per-cell RNA content doubling

## Inputs

- Phase-annotated h5ad files with adata.obs['cell_cycle_theta'] from a DeepCycle run (mESC, human fibroblast, ductal cell progenitor)
- Total UMI counts per cell from the h5ad count matrix
- numpy/scipy Python environment (numpy 1.19.1, scipy 1.5.2 per article supplementary)

## Outputs

- UMI-vs-phase binned profile (mean total counts per cell per cell_cycle_theta bin)
- Exponential doubling fit over the pre-mitotic phase interval
- umi_fold_changes.csv with fold changes per dataset (expected ~2.1 mESCs, ~2.0 ductal, ~2.1 fibroblasts)

## How to apply

Download the phase-annotated h5ad files (mESCs, human fibroblasts from Zenodo 10.5281/zenodo.4719436; ductal cell progenitors from the public dataset), each carrying obs['cell_cycle_theta'] from a DeepCycle run. Bin cells circularly by cell_cycle_theta into phase bins spanning the full cycle, sum total UMI counts per cell, and average them within each phase bin to obtain the UMI-vs-phase profile. Fit an exponential doubling trend with numpy/scipy to the binned UMI counts over the phase interval preceding the mitotic drop. Compute the fold change as the peak preceding mitosis relative to the preceding minimum. Write the three fold changes to umi_fold_changes.csv; the doubling rationale is that the mother cell approximately doubles in volume (and RNA) before division.

## Related tools

- **DeepCycle** (Produced the circular cell_cycle_theta annotation in the h5ad files that defines the phase axis for binning cells) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the pp.moments preprocessing (Ms/Mu layers) required upstream of DeepCycle; preprocessed anndata is the DeepCycle input) — https://scvelo.readthedocs.io/
- **numpy** (Circular binning of cells by cell_cycle_theta and computation of per-bin mean UMI counts)
- **scipy** (Curve fitting of the exponential doubling trend to the binned UMI counts)

## Examples

```
from scipy.optimize import curve_fit; import numpy as np; fold = curve_fit(lambda t,a,b: a*np.exp(b*t), theta_bins, umi_bins)[0][0] * (np.exp(...) )  # fit UMI ~ a*exp(b*theta) over pre-mitotic bins
```

## Evaluation signals

- Computed fold change per dataset reproduces the reported values of 2.1 (mESCs), 2.0 (ductal), and 2.1 (fibroblasts) within a small tolerance
- The fitted exponential reaches its peak in the phase bin immediately preceding the mitotic drop, with a clear minimum in the preceding segment
- Binned UMI-vs-phase profile is continuous across the circular wrap-around of cell_cycle_theta (no artificial discontinuity at 0/2π)
- umi_fold_changes.csv contains exactly three entries, one per dataset, matching the expected order and ranges

## Limitations

- Fold change depends on sequencing depth; the README warns that if no cycling genes are found results are unreliable and deeper sequencing (more RNAs per cell) may be required
- For human fibroblasts the analysis must be restricted to the leiden cluster 0 subpopulation; using the full mixed population distorts the phase-resolved UMI profile
- Fitting is sensitive to where the pre-mitotic doubling segment is delimited relative to the M/G1 transition

## Evidence

- [readme] cell_cycle_theta availability: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes"
- [readme] phase-annotated data source: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [abstract] biological expectation of G1-phase RNA changes: "Characterizing the cell cycle in embryonic and somatic cells, we identify major waves of transcription during the G1 phase"
- [readme] unreliable-run caveat: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper"
