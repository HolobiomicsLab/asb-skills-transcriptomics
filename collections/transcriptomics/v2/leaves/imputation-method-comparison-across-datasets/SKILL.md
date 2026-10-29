---
name: imputation-method-comparison-across-datasets
description: Use when you have imputed scRNA-seq matrices from multiple methods on
  log10(count+1) scale plus ground truth, and need to visually compare which methods
  recover nonlinear gene-gene patterns for selected gene pairs.
when_to_use_negative:
- You only have raw count data and no imputation results — run LATE or TRANSLATE imputation
  first (`late`, `pre-training` + `translate` modes) before comparing
- Ground truth does not exist for your data set and you cannot set it to the input
  — comparison against truth becomes circular (as in the mouse brain example where
  ground truth is set to the input)
- Your matrices are on a different scale or have mismatched gene/cell identifiers
  — mixing scales would produce misleading scatterplots
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: LATE
  role: Autoencoder-based imputation method whose recovered nonlinear gene-gene relationships
    are visualized in the comparison
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: Transfer-learning variant of LATE (pre-training on reference data, then translate
    on input data) whose results are compared in the same panels
  repo: https://github.com/audreyqyfu/LATE
- name: pandas
  role: Reading hd5 imputed matrices with the tables backend for plotting
- name: matplotlib
  role: Drawing the per-method scatterplot panels of gene-gene expression
- name: seaborn
  role: Styling scatterplot panels to mirror the published figures
- name: tables
  role: Python module for reading the hd5-format imputed output files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:60890d0041700330804274b3a44341112796363fe13eed6c697dac7240348fe4
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# imputation-method-comparison-across-datasets

## Summary

Reproduce side-by-side scatterplots of nonlinear gene-gene relationships recovered by LATE and TRANSLATE versus baseline imputation methods (MAGIC, DCA, scVI, SAVER, ALRA, scImpute) and the ground truth, on imputed scRNA-seq matrices such as MAGIC_mouse, GTEx_4tissues, and PBMC_G949.

## When to use

Use when you have imputed scRNA-seq matrices from multiple methods on log10(count+1) scale plus ground truth, and need to visually compare which methods recover nonlinear gene-gene patterns for selected gene pairs.

## When NOT to use

- You only have raw count data and no imputation results — run LATE or TRANSLATE imputation first (`late`, `pre-training` + `translate` modes) before comparing
- Ground truth does not exist for your data set and you cannot set it to the input — comparison against truth becomes circular (as in the mouse brain example where ground truth is set to the input)
- Your matrices are on a different scale or have mismatched gene/cell identifiers — mixing scales would produce misleading scatterplots

## Inputs

- LATE/TRANSLATE imputed matrices in hd5 format (`step2/imputation.step2.hd5`, log10(count+1) scale)
- Baseline imputed matrices from MAGIC, SAVER, ALRA, scImpute (and optionally DCA, scVI) on the same data sets
- Ground-truth expression matrices for MAGIC_mouse, GTEx_4tissues, and PBMC_G949 data sets (same format as input)
- Gene-pair selections corresponding to Figures 2-4 of the paper

## Outputs

- PNG scatterplot figures per data set (e.g., `Eval/gene_pair_scatterplots.png`) with one panel per method plus ground truth, mirroring Figures 2-4
- Qualitative comparison showing LATE/TRANSLATE panels with sharply defined nonlinear gene-gene patterns versus blurred baseline panels

## How to apply

Load the LATE and TRANSLATE imputed matrices from `step2/imputation.step2.hd5` (hd5 format, log10(count+1) scale) using pandas with the tables module, together with baseline imputed matrices (e.g., MAGIC, SAVER, ALRA, scImpute, DCA, scVI) and the ground-truth matrices for each data set. Verify all matrices share the gene and cell identifiers of the input data and are on the same log10(count+1) scale before plotting — LATE/TRANSLATE outputs have the same layout as the input and are already on this scale. Select the gene pairs highlighted in Figures 2-4 for MAGIC_mouse, GTEx_4tissues, and PBMC_G949. Using matplotlib and seaborn, draw scatterplots of the two genes' imputed expression across cells, one panel per method plus ground truth, arranged to mirror the published figure layout, and save as PNG (e.g., `Eval/gene_pair_scatterplots.png` per data set). The `analysis` mode of the LATE scripts generates this `Eval/` folder from results in `step2/`.

## Related tools

- **LATE** (Autoencoder-based imputation method whose recovered nonlinear gene-gene relationships are visualized in the comparison) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning variant of LATE (pre-training on reference data, then translate on input data) whose results are compared in the same panels) — https://github.com/audreyqyfu/LATE
- **pandas** (Reading hd5 imputed matrices with the tables backend for plotting)
- **matplotlib** (Drawing the per-method scatterplot panels of gene-gene expression)
- **seaborn** (Styling scatterplot panels to mirror the published figures)
- **tables** (Python module for reading the hd5-format imputed output files)

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Each reproduced panel qualitatively matches the corresponding published figure (Figures 2-4 layout)
- LATE and TRANSLATE panels show the sharply defined nonlinear gene-gene relationship in MAGIC_mouse data
- MAGIC itself and DCA, scVI, SAVER, ALRA, scImpute panels show the blurred pattern reported in the paper rather than the recovered pattern
- All plotted matrices share the same gene and cell identifiers as the input data and are on the log10(count+1) scale
- Output PNGs are written to the Eval/ folder generated by the `analysis` mode

## Limitations

- Comparison is qualitative (visual pattern recovery); the paper's quantitative evaluation uses mean squared error separately
- LATE and TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value, so biological zeros are not distinguished from technical zeros
- The comparison depends on the specific gene pairs highlighted in Figures 2-4; results may not generalize to arbitrary gene pairs
- LATE is not compatible with TensorFlow 2; a virtual environment with TensorFlow 1.x (e.g., 1.15) and pandas 0.25.1 is needed to generate the imputation results

## Evidence

- [abstract] nonlinear gene-gene recovery finding: [evidence span withheld — anchor status: not_found]
