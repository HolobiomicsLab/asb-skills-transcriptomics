---
name: single-cell-imputation-result-visualization
description: Use when imputed single-cell expression matrices (hd5, log10(count+1)
  scale) are available and you need to visually assess whether an imputation method
  recovers nonlinear gene-gene relationships or compare methods against ground truth.
when_to_use_negative:
- When no imputation results exist yet — the analysis mode requires results already
  stored in the step2/ folder
- When the data are raw un-imputed counts and the goal is imputation itself rather
  than visual validation of imputed results (use the late/pre-training/translate modes
  instead)
- When a quantitative accuracy metric (e.g. mean squared error against ground truth)
  is the required deliverable rather than qualitative gene-pair pattern recovery
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: pandas
  role: Read the hd5 imputed matrices (with tables) and align gene/cell identifiers
    before plotting
- name: tables
  role: Backend for reading the hd5-format imputed output (step2/imputation.step2.hd5)
- name: matplotlib
  role: Draw the per-method gene-pair scatterplot panels
- name: seaborn
  role: Styling and layout of the multi-panel scatterplots mirroring Figures 2-4
- name: numpy
  role: Array handling of imputed and ground-truth matrices on the log10(count+1)
    scale
- name: LATE
  role: Source of the primary imputed matrices being visualized (step2/imputation.step2.hd5)
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: Source of transfer-learning imputed matrices being visualized alongside LATE
  repo: https://github.com/audreyqyfu/LATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:128972ab7a6f2545a8cf1250a86e2d04195c1a4dd7942ee81f945ed9da4d4a1f
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# single-cell-imputation-result-visualization

## Summary

Visualizes and validates scRNA-seq imputation results by drawing gene-pair scatterplots of imputed expression (log10(count+1)) for LATE, TRANSLATE, and baseline methods (MAGIC, DCA, scVI, SAVER, ALRA, scImpute) alongside ground truth, to show whether nonlinear gene-gene relationships are recovered.

## When to use

Use when imputed single-cell expression matrices (hd5, log10(count+1) scale) are available and you need to visually assess whether an imputation method recovers nonlinear gene-gene relationships or compare methods against ground truth.

## When NOT to use

- When no imputation results exist yet — the analysis mode requires results already stored in the step2/ folder
- When the data are raw un-imputed counts and the goal is imputation itself rather than visual validation of imputed results (use the late/pre-training/translate modes instead)
- When a quantitative accuracy metric (e.g. mean squared error against ground truth) is the required deliverable rather than qualitative gene-pair pattern recovery

## Inputs

- Imputed data matrix in hd5 format from step2/imputation.step2.hd5 (log10(count+1) scale)
- Baseline imputed matrices from MAGIC, DCA, scVI, SAVER, ALRA, and scImpute
- Ground-truth expression matrix (e.g. example.hd5 for PBMC_G949_10K), same format as input
- List of gene pairs highlighted in the paper's Figures 2-4 for MAGIC_mouse, GTEx_4tissues, and PBMC_G949

## Outputs

- PNG scatterplot figures of gene-pair expression, one panel per method plus ground truth (e.g. Eval/gene_pair_scatterplots.png per data set)
- Eval/ folder with summary and visualization outputs from the analysis mode

## How to apply

Load the imputed matrices from `step2/imputation.step2.hd5` (LATE or TRANSLATE output, log10(count+1) scale, hd5 format) using pandas/tables, together with baseline imputed matrices (MAGIC, SAVER, ALRA, scImpute, DCA, scVI) and the ground-truth matrix for the same data set (e.g. MAGIC_mouse, GTEx_4tissues, PBMC_G949). Verify all matrices are on the same log10(count+1) scale and share gene and cell identifiers of the input before plotting. Select the gene pairs highlighted in the paper's figures (Figures 2-4 for the three data sets), then use matplotlib and seaborn to draw per-method scatterplot panels of the two genes' expression across cells, arranged to mirror the published figure layout. Save each figure as PNG (e.g. `Eval/gene_pair_scatterplots.png` per data set); the `analysis` mode of the LATE pipeline generates the `Eval/` folder for such outputs.

## Related tools

- **pandas** (Read the hd5 imputed matrices (with tables) and align gene/cell identifiers before plotting)
- **tables** (Backend for reading the hd5-format imputed output (step2/imputation.step2.hd5))
- **matplotlib** (Draw the per-method gene-pair scatterplot panels)
- **seaborn** (Styling and layout of the multi-panel scatterplots mirroring Figures 2-4)
- **numpy** (Array handling of imputed and ground-truth matrices on the log10(count+1) scale)
- **LATE** (Source of the primary imputed matrices being visualized (step2/imputation.step2.hd5)) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Source of transfer-learning imputed matrices being visualized alongside LATE) — https://github.com/audreyqyfu/LATE

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Each reproduced panel qualitatively matches the corresponding published figure (Figures 2-4 layout and appearance)
- LATE/TRANSLATE panels show the sharply defined nonlinear gene-gene pattern while baseline (MAGIC, DCA, scVI, SAVER, ALRA, scImpute) panels show the blurred pattern reported in the paper
- All plotted matrices are verified to be on the same log10(count+1) scale and share the input data's gene and cell identifiers
- Output figures are saved as PNG files in the Eval/ folder generated by the analysis mode (e.g. Eval/gene_pair_scatterplots.png per data set)
- Ground-truth panel values match the ground-truth matrix (e.g. example.hd5 for PBMC_G949_10K), not the masked input

## Limitations

- The comparison is qualitative; pattern recovery in scatterplots is judged by visual matching against published figures rather than a numeric criterion
- Reproducing the exact published panels requires the gene pairs highlighted in Figures 2-4 and the baseline imputed matrices from all compared methods (MAGIC, DCA, scVI, SAVER, ALRA, scImpute), which must be generated separately
- LATE/TRANSLATE impute all zeros with nonzero values assuming they are technical dropouts, which shapes the visualized patterns; biological zeros are not distinguished

## Evidence
