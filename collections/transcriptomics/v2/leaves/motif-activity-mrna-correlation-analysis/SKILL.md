---
name: motif-activity-mrna-correlation-analysis
description: Use when you need to infer TF motif activities from scRNA-seq unspliced
  promoter counts and test whether a TF's motif activity tracks its own mRNA level
  across cell cycle phases.
when_to_use_negative:
- The dataset lacks an unspliced layer — the model is fit on unspliced (Mu) counts,
  so spliced-only matrices cannot support motif activity inference
- No motif/PWM site annotations (e.g. Swiss Regulon GFF) exist for the organism, so
  the promoter-by-motif matrix cannot be built
- The population is not cycling — if no cycling genes are found, DeepCycle theta and
  downstream motif-activity dynamics will not be reliable
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scVelo
  role: Provides the imputed unspliced 'Mu' and spliced 'Ms' moments used to build
    the promoter-by-motif count matrix
  repo: https://scvelo.readthedocs.io/
- name: DeepCycle
  role: Assigns a continuous cell cycle trajectory (obs['cell_cycle_theta']) to ductal
    cells so motif activity and mRNA can be compared across G1 to M
  repo: https://github.com/andreariba/DeepCycle
- name: numpy
  role: Numerical backend for the ridge-regularized ISMARA linear model fit on unspliced
    counts
- name: scipy
  role: Computes the Pearson correlation and the exact test of no-correlation between
    motif activity and mRNA level
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:ae02044eec979aa317fc36d12e61dcf03148eb57340bd9248bb6ce1dab9e7d45
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCycle
  - Python
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# motif-activity-mrna-correlation-analysis

## Summary

Reproduces an ISMARA-like transcription factor motif activity analysis on single-cell RNA-seq unspliced counts and quantifies the correlation between a TF's motif activity and its mRNA abundance across the cell cycle (e.g. Ybx1 in ductal cell progenitors, r = 0.91, p = 4e-19).

## When to use

Use when you need to infer TF motif activities from scRNA-seq unspliced promoter counts and test whether a TF's motif activity tracks its own mRNA level across cell cycle phases.

## When NOT to use

- The dataset lacks an unspliced layer — the model is fit on unspliced (Mu) counts, so spliced-only matrices cannot support motif activity inference
- No motif/PWM site annotations (e.g. Swiss Regulon GFF) exist for the organism, so the promoter-by-motif matrix cannot be built
- The population is not cycling — if no cycling genes are found, DeepCycle theta and downstream motif-activity dynamics will not be reliable

## Inputs

- Public ductal cell progenitor scRNA-seq dataset (h5ad with 'Mu'/'Ms' layers from scVelo pp.moments)
- Swiss Regulon mm10/hg19 GFF files of Motevo-predicted TF binding sites
- GOterm cell_cycle gene list (e.g. GO_cell_cycle_annotation_mouse.txt) for cycling-gene selection
- cell_cycle_theta per cell from DeepCycle to order cells across G1 to M

## Outputs

- Promoter-by-motif count matrix restricted to TSS +/- 1 kb promoter windows
- Per-cell motif activity table estimated with the ridge-regularized ISMARA linear model
- Pearson correlation between Ybx1 motif activity and Ybx1 mRNA abundance (r = 0.91, p = 4e-19 exact test of no-correlation)
- Cross-validated ridge penalty value (10-fold CV over cells)

## How to apply

Download Swiss Regulon mm10/hg19 GFF files with Motevo-predicted TF binding sites and restrict annotations to promoter windows spanning TSS +/- 1 kb. Load the public ductal cell progenitor scRNA-seq dataset and extract unspliced read counts (the scVelo 'Mu' layer) to build a promoter-by-motif count matrix. Fit the ISMARA linear model on the unspliced read counts with ridge regularization, tuning the ridge penalty by 10-fold cross-validation over cells. Compute the Pearson correlation between Ybx1 motif activity and Ybx1 mRNA abundance across ductal cells, ordered along the cell cycle via DeepCycle's cell_cycle_theta.

## Related tools

- **scVelo** (Provides the imputed unspliced 'Mu' and spliced 'Ms' moments used to build the promoter-by-motif count matrix) — https://scvelo.readthedocs.io/
- **DeepCycle** (Assigns a continuous cell cycle trajectory (obs['cell_cycle_theta']) to ductal cells so motif activity and mRNA can be compared across G1 to M) — https://github.com/andreariba/DeepCycle
- **numpy** (Numerical backend for the ridge-regularized ISMARA linear model fit on unspliced counts)
- **scipy** (Computes the Pearson correlation and the exact test of no-correlation between motif activity and mRNA level)

## Examples

```
import scvelo as scv; scv.pp.moments(adata)  # compute imputed 'Ms'/'Mu' layers used for the promoter-by-motif count matrix
```

## Evaluation signals

- The reproduced Ybx1 motif-activity vs mRNA correlation matches the reported r = 0.91 (p = 4e-19 exact test, Fig. 4B) within tolerance
- Both Ybx1 motif activity and Ybx1 mRNA increase monotonically from G1 to M in ductal cells
- The motif activity table is complete for all promoters in the restricted TSS +/- 1 kb window set
- The ridge penalty is selected via 10-fold cross-validation over cells rather than fixed arbitrarily
- Input adata contains scVelo 'Ms' and 'Mu' layers and obs['cell_cycle_theta'] before running the analysis

## Limitations

- If no cycling genes are found, results are unreliable — sequencing may need to be deeper (more RNAs per cell)
- Promoter activity inference is restricted to TSS +/- 1 kb windows, so distal enhancer-driven TF effects are not captured
- Defining cell cycle phases in RNA data based solely on gene markers lacks generalizability across cellular models; dynamical features (cell_cycle_theta) should be used instead

## Evidence

- [other] In ductal cells, Ybx1 mRNA level and its motif activity show a very high correlation (r = 0.91, p = 4e-19 exact test of no-correlation): [evidence span withheld — anchor status: not_found]
- [supplementary] the layers 'Ms' and 'Mu' from scVelo: "the layers 'Ms' and 'Mu' from scVelo"
- [readme] The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [intro] extend the analysis to a public dataset of ductal cell progenitors: "but also extend the analysis to a public dataset of ductal cell progenitors"
