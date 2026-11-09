---
name: dropout-recovery-assessment
description: Use when you have imputed single-cell expression matrices (hd5, log10(count+1)
  scale) plus a ground truth, and need to verify that dropout imputation recovers
  nonlinear gene-gene patterns rather than blurring them.
when_to_use_negative:
- No ground-truth (or reference) matrix exists for the dataset — the paper sets ground
  truth equal to the input in that case, so the recovery assessment is uninformative
- The imputed matrices are on a different scale than log10(count+1) or lack shared
  gene/cell identifiers with the input — panels would be incomparable until harmonized
- The goal is quantitative imputation accuracy (e.g. mean squared error comparison)
  rather than visual assessment of recovered nonlinear gene-gene relationships
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: LATE
  role: Autoencoder-based imputation whose output matrix is one of the methods assessed
    for nonlinear gene-gene relationship recovery
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: Transfer-learning variant of LATE whose imputed matrix is assessed alongside
    LATE
  repo: https://github.com/audreyqyfu/LATE
- name: pandas
  role: Loading the hd5 imputed matrices (with the tables module) for plotting
- name: matplotlib
  role: Drawing the per-method gene-pair scatterplot panels
- name: seaborn
  role: Styling the scatterplot panels to mirror Figures 2-4
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:dda15ef46e9ef8cb2b299bd3b8fd939461ea94ac1f592245c5dd6b86e9501919
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - LATE
  - Python+
  - TRANSLATE
  - TensorFlow
  - matplotlib
  - numpy
  - pandas
  - seaborn
  - tables
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# dropout-recovery-assessment

## Summary

Assesses whether imputation methods (LATE, TRANSLATE) recover nonlinear gene-gene relationships masked by scRNA-seq dropouts, by drawing scatterplots of gene-pair expression from imputed matrices (LATE, TRANSLATE, MAGIC, SAVER, ALRA, scImpute, DCA, scVI) against the ground truth on datasets such as MAGIC_mouse, GTEx_4tissues, and PBMC_G949.

## When to use

Use when you have imputed single-cell expression matrices (hd5, log10(count+1) scale) plus a ground truth, and need to verify that dropout imputation recovers nonlinear gene-gene patterns rather than blurring them.

## When NOT to use

- No ground-truth (or reference) matrix exists for the dataset — the paper sets ground truth equal to the input in that case, so the recovery assessment is uninformative
- The imputed matrices are on a different scale than log10(count+1) or lack shared gene/cell identifiers with the input — panels would be incomparable until harmonized
- The goal is quantitative imputation accuracy (e.g. mean squared error comparison) rather than visual assessment of recovered nonlinear gene-gene relationships

## Inputs

- LATE/TRANSLATE imputed matrix in hd5 format at `step2/imputation.step2.hd5` (log10(count+1) scale)
- Baseline imputed matrices from MAGIC, SAVER, ALRA, scImpute (and optionally DCA, scVI)
- Ground-truth expression matrix for the target dataset (e.g. MAGIC_mouse, GTEx_4tissues, PBMC_G949)
- List of gene pairs of interest (those highlighted in Figures 2-4)

## Outputs

- Scatterplot figures of gene-pair imputed expression across cells, one panel per method plus ground truth (e.g. `Eval/gene_pair_scatterplots.png` per dataset)

## How to apply

Load the LATE and TRANSLATE imputed matrices from `step2/imputation.step2.hd5` (hd5 format, log10(count+1) scale) using pandas with the tables module, alongside baseline imputations from MAGIC, SAVER, ALRA, scImpute (and optionally DCA, scVI) and the ground-truth matrices. First verify that all matrices are on the same log10(count+1) scale and share gene and cell identifiers with the input data — the LATE pipeline transforms inputs with log10(count+1) before imputation and stores outputs in the same layout. Select the gene pairs highlighted in the paper's Figures 2-4 for each dataset (MAGIC_mouse, GTEx_4tissues, PBMC_G949). Using matplotlib and seaborn, draw scatterplots of the two genes' imputed expression across cells, one panel per method plus ground truth, arranged to mirror the published figure layout. The assessment succeeds if LATE/TRANSLATE panels show the sharply defined nonlinear gene-gene pattern while baseline panels show the blurred pattern reported in the paper; validation is qualitative visual comparison against the published figures.

## Related tools

- **LATE** (Autoencoder-based imputation whose output matrix is one of the methods assessed for nonlinear gene-gene relationship recovery) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning variant of LATE whose imputed matrix is assessed alongside LATE) — https://github.com/audreyqyfu/LATE
- **pandas** (Loading the hd5 imputed matrices (with the tables module) for plotting)
- **matplotlib** (Drawing the per-method gene-pair scatterplot panels)
- **seaborn** (Styling the scatterplot panels to mirror Figures 2-4)

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Each reproduced scatterplot panel qualitatively matches the corresponding published figure (Figures 2-4) for the dataset
- LATE and TRANSLATE panels show the sharply defined nonlinear gene-gene pattern; MAGIC, DCA, scVI, SAVER, ALRA and scImpute panels show the blurred pattern reported in the paper
- All matrices are confirmed to be on the log10(count+1) scale and share gene and cell identifiers with the input before plotting
- Output figures are saved as PNG files (e.g. `Eval/gene_pair_scatterplots.png`) per dataset, mirroring the paper's layout

## Limitations

- LATE and TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value; biological vs technical zeros are not distinguished
- Validation of the recovered nonlinear pattern is qualitative (visual match to published figures), not a quantitative metric
- The current LATE code needs to run where it is stored on your computer; LATE is not compatible with TensorFlow 2

## Evidence

- [readme] hd5 output on log10(count+1) scale: "Imputed data matrix is in the hd5 format and stored in `step2/imputation.step2.hd5`."
- [readme] input transformation: "The input data will be transformed with log10(count+1) for imputation."
- [readme] analysis mode: "`analysis`:
  - Uses imputation results in `step2` for summary and visualization;
  - Generates folder `Eval/`."
- [abstract] finding: "recovering nonlinear gene-gene relationships, and better separating cell types"
