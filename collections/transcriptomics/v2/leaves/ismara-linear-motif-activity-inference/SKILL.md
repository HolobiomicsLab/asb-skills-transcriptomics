---
name: ismara-linear-motif-activity-inference
description: Use when you need to infer TF motif activity trajectories across single-cell
  cell-cycle phases from unspliced scRNA-seq counts and promoter-annotated motif binding
  sites.
when_to_use_negative:
- 'Input counts are spliced-only (Ms layer): the model is fit on unspliced ''Mu''
  counts to capture transcriptional dynamics'
- No cycling genes/patterns are found in the dataset — DeepCycle results would be
  unreliable and downstream motif activity correlations meaningless
- No Motevo/Swiss Regulon motif annotation GFF is available for the organism — the
  promoter-by-motif matrix cannot be constructed
tools:
- name: DeepCycle
  role: Annotates each cell with the continuous cell_cycle_theta used to order cells
    and correlate motif activity with cell-cycle phase
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Computes the imputed unspliced 'Mu' layer that serves as the count input for
    the promoter-by-motif matrix
- name: numpy
  role: Numerical construction of the promoter-by-motif matrix and ridge-regularized
    model fitting
- name: scipy
  role: Computes the Pearson correlation and exact test of no-correlation between
    Ybx1 motif activity and mRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:4ee1636d6bc6298e7ca9e4d6d071f157390f050f2ff0e3fb3d0d6d6c2dc91c17
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
yes_rate: 0.7
n_claims: 10
---

# ismara-linear-motif-activity-inference

## Summary

Reproduces ISMARA-style transcription factor motif activity estimation by fitting a ridge-regularized linear model on scVelo 'Mu' (unspliced) read counts aggregated over Motevo-predicted promoter binding sites, enabling motif activity dynamics across the cell cycle.

## When to use

Use when you need to infer TF motif activity trajectories across single-cell cell-cycle phases from unspliced scRNA-seq counts and promoter-annotated motif binding sites.

## When NOT to use

- Input counts are spliced-only (Ms layer): the model is fit on unspliced 'Mu' counts to capture transcriptional dynamics
- No cycling genes/patterns are found in the dataset — DeepCycle results would be unreliable and downstream motif activity correlations meaningless
- No Motevo/Swiss Regulon motif annotation GFF is available for the organism — the promoter-by-motif matrix cannot be constructed

## Inputs

- Swiss Regulon mm10/hg19 GFF files of Motevo-predicted TF binding sites
- Ductal cell progenitor scRNA-seq h5ad Anndata preprocessed with velocyto and scvelo.pp.moments (Ms/Mu layers)
- DeepCycle-annotated obs['cell_cycle_theta'] per cell
- Ybx1 mRNA abundance per ductal cell

## Outputs

- Promoter-by-motif count matrix from TSS +/- 1 kb windows
- Ridge-regularized motif activity estimates per cell (with 10-fold CV-tuned penalty)
- Pearson correlation between Ybx1 motif activity and Ybx1 mRNA (expected r = 0.91, p = 4e-19)

## How to apply

Download the Swiss Regulon mm10/hg19 GFF files with Motevo-predicted transcription factor binding sites and restrict annotations to promoter windows spanning the TSS +/- 1 kb. Load the ductal cell progenitor scRNA-seq dataset (preprocessed with velocyto, scvelo moments, and DeepCycle) and extract the unspliced 'Mu' layer to build a promoter-by-motif count matrix. Fit the ISMARA linear model on the unspliced counts with ridge regularization, tuning the penalty via 10-fold cross-validation over cells. Finally, compute the Pearson correlation between Ybx1 motif activity and Ybx1 mRNA abundance across ductal cells to recover cell-cycle dynamics that monotonically increase from G1 to M.

## Related tools

- **DeepCycle** (Annotates each cell with the continuous cell_cycle_theta used to order cells and correlate motif activity with cell-cycle phase) — https://github.com/andreariba/DeepCycle
- **scVelo** (Computes the imputed unspliced 'Mu' layer that serves as the count input for the promoter-by-motif matrix)
- **numpy** (Numerical construction of the promoter-by-motif matrix and ridge-regularized model fitting)
- **scipy** (Computes the Pearson correlation and exact test of no-correlation between Ybx1 motif activity and mRNA)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(ybx1_motif_activity, ybx1_mrna)
```

## Evaluation signals

- Reproduced Ybx1 correlation matches the reported r = 0.91 (p = 4e-19, exact test of no-correlation, Fig. 4B) within tolerance
- The motif activity table is complete for all promoters in the TSS +/- 1 kb restricted annotation
- Ybx1 motif activity and mRNA are both monotonically increasing from G1 to M phase
- Ridge penalty converges in the 10-fold cross-validation over cells without numerical failures

## Limitations

- If no cycling genes are detected (e.g. insufficient sequencing depth / RNAs per cell), the downstream cell-cycle trajectory and motif activity correlations will not be reliable
- Relies on the quality of Motevo-predicted binding-site annotations and the promoter window restriction (TSS +/- 1 kb); motif activity estimates inherit promoter-annotation biases
- Ridge regularization strength must be cross-validated; a fixed penalty may bias motif activity dynamics

## Evidence

- [other] ISMARA linear model on unspliced counts with ridge regularization: [evidence span withheld — anchor status: not_found]
- [other] Promoter window restriction to TSS +/- 1 kb: [evidence span withheld — anchor status: not_found]
- [other] Reported Ybx1 correlation: [evidence span withheld — anchor status: not_found]
- [supplementary] Unspliced layer source: "the layers 'Ms' and 'Mu' from scVelo"
- [readme] Unreliable results without cycling genes: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
