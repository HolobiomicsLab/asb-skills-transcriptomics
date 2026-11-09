---
name: ridge-regression-cross-validation
description: Use when you must estimate TF motif activities from a promoter-by-motif
  count matrix built on unspliced scRNA-seq reads (scVelo 'Mu'), and the ridge penalty
  needs to be tuned by cross-validation so motif activities can be correlated with
  mRNA abundance across cell-cycle phases.
when_to_use_negative:
- Input data are spliced counts ('Ms') or raw gene expression rather than scVelo imputed
  unspliced reads — the ISMARA fit here is defined on the unspliced 'Mu' layer
- 'The dataset has no reliable cycling signal: if no cycling genes are found (e.g.
  insufficient sequencing depth), the cell-cycle ordering that motif activities are
  correlated against is unreliable'
- The goal is de novo motif discovery from sequence, rather than estimating the activities
  of pre-defined Motevo/Swiss Regulon motifs over promoter windows
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepCycle
  role: Assigns obs['cell_cycle_theta'] to each cell and consumes the scVelo moments,
    providing the trajectory along which motif activities are ordered and correlated
  repo: github.com/andreariba/DeepCycle
- name: scVelo
  role: Computes the imputed unspliced 'Mu' and spliced 'Ms' layers (pp.moments) used
    as the response for the ridge-regularized motif activity model
  repo: https://scvelo.readthedocs.io/
- name: numpy
  role: Linear-algebra backend for building the promoter-by-motif count matrix and
    solving the ridge-regularized fit
- name: scipy
  role: Pearson correlation and exact test of no-correlation between Ybx1 motif activity
    and Ybx1 mRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:32f28cc7e13dffe265ae8407aac8afca6e135649dfa3fd808c19e4e314c2fb84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCycle
  - Python
  - numpy
  - scVelo
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ridge-regression-cross-validation

## Summary

Fit an ISMARA-like linear model on scVelo imputed unspliced read counts (the 'Mu' layer) to estimate transcription factor motif activities, using ridge regularization whose penalty is tuned by 10-fold cross-validation over cells. This underpins quantifying, e.g., the Ybx1 motif-activity dynamics along the DeepCycle cell-cycle trajectory in ductal cell progenitors.

## When to use

Use when you must estimate TF motif activities from a promoter-by-motif count matrix built on unspliced scRNA-seq reads (scVelo 'Mu'), and the ridge penalty needs to be tuned by cross-validation so motif activities can be correlated with mRNA abundance across cell-cycle phases.

## When NOT to use

- Input data are spliced counts ('Ms') or raw gene expression rather than scVelo imputed unspliced reads — the ISMARA fit here is defined on the unspliced 'Mu' layer
- The dataset has no reliable cycling signal: if no cycling genes are found (e.g. insufficient sequencing depth), the cell-cycle ordering that motif activities are correlated against is unreliable
- The goal is de novo motif discovery from sequence, rather than estimating the activities of pre-defined Motevo/Swiss Regulon motifs over promoter windows

## Inputs

- Anndata (h5ad) preprocessed with velocyto and scvelo.pp.moments, containing imputed unspliced ('Mu') and spliced ('Ms') layers and obs['cell_cycle_theta'] from DeepCycle
- Swiss Regulon mm10/hg19 GFF files of Motevo-predicted TF binding sites, restricted to promoter windows TSS +/- 1 kb
- Promoter-by-motif count matrix over cells (unspliced counts aggregated per promoter per motif)
- Per-cell Ybx1 (target TF) mRNA abundance for correlation analysis

## Outputs

- Per-cell motif activity table for all promoters (ridge-regularized ISMARA fit)
- Selected ridge penalty (lambda) from 10-fold cross-validation over cells
- Pearson correlation coefficient and exact-test p-value between Ybx1 motif activity and Ybx1 mRNA (expected r = 0.91, p = 4e-19)
- Motif activity dynamics ordered along the cell-cycle trajectory (G1 to M)

## How to apply

Build a promoter-by-motif count matrix from Swiss Regulon mm10/hg19 GFF Mototevo-predicted binding sites restricted to promoter windows spanning TSS +/- 1 kb, with the response being imputed unspliced read counts (scVelo 'Mu' layer, obtained via scvelo.pp.moments on the velocyto-annotated anndata). Fit the ISMARA linear model on the unspliced counts with ridge regularization; because cells share promoter composition, ridge stabilizes the motif-activity coefficients. Tune the ridge penalty via 10-fold cross-validation over the cells (holding out cells, not promoters), then refit on all cells at the selected penalty. For validation, compute the Pearson correlation between Ybx1 motif activity and Ybx1 mRNA abundance across ductal cells and check that motif activity increases monotonically from G1 to M, consistent with Ybx1 positively regulating G1 and G2/M.

## Related tools

- **DeepCycle** (Assigns obs['cell_cycle_theta'] to each cell and consumes the scVelo moments, providing the trajectory along which motif activities are ordered and correlated) — github.com/andreariba/DeepCycle
- **scVelo** (Computes the imputed unspliced 'Mu' and spliced 'Ms' layers (pp.moments) used as the response for the ridge-regularized motif activity model) — https://scvelo.readthedocs.io/
- **numpy** (Linear-algebra backend for building the promoter-by-motif count matrix and solving the ridge-regularized fit)
- **scipy** (Pearson correlation and exact test of no-correlation between Ybx1 motif activity and Ybx1 mRNA)

## Examples

```
from sklearn.linear_model import RidgeCV; model = RidgeCV(alphas=np.logspace(-3, 3, 13), cv=10).fit(motif_counts_TSSpm1kb, mu_unspliced)
```

## Evaluation signals

- 10-fold cross-validation over cells converges on a stable ridge penalty; refitting at that penalty yields a motif activity table complete for all promoters
- Reproduced Ybx1 motif-activity vs mRNA correlation matches the reported r = 0.91 (p = 4e-19, exact test of no-correlation, Fig. 4B) within tolerance
- Ybx1 motif activity increases monotonically from G1 to M, consistent with Ybx1 positively regulating the G1 and G2/M phases
- Input anndata contains the required scVelo layers 'Ms' and 'Mu' and obs['cell_cycle_theta'] before fitting
- Held-out CV loss at the chosen ridge penalty is at its minimum across the scanned penalty grid (no under-/over-fitting of motif activities)

## Limitations

- If no cycling genes are detected the downstream cell-cycle ordering and motif-activity correlations are unreliable; deeper sequencing (more RNAs per cell) may be required
- Fitting on imputed counts (scVelo moments) smooths single-cell variability, so activity estimates depend on the moments preprocessing and the promoter window choice (TSS +/- 1 kb)
- The ridge penalty is tuned over cells; correlations tied to a specific cell type (ductal cell progenitors, r = 0.91) may not transfer to other systems without re-tuning

## Evidence

- [methods] task_007 workflow: ISMARA ridge fit: [evidence span withheld — anchor status: not_found]
- [methods] task_007 workflow: penalty tuning: [evidence span withheld — anchor status: not_found]
- [results] task_007 workflow: correlation validation: [evidence span withheld — anchor status: not_found]
- [readme] scVelo moments preprocessing requirement: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [supplementary] Required anndata layers for downstream fitting: "The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo."
