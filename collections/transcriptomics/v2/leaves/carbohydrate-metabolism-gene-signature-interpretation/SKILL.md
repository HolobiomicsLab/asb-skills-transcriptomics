---
name: carbohydrate-metabolism-gene-signature-interpretation
description: Use when a reconstructed scorecard from S. mutans (SMU) transcriptomic
  comparisons under different dietary carbohydrates must be annotated/validated against
  known PEP-PTS and carbohydrate-metabolism gene families.
when_to_use_negative:
- Input data are not transcriptomic comparisons under different carbohydrate/dietary
  conditions (e.g. antibiotic exposure datasets 1-2 or metal-surface dataset 4), where
  PTS/carbohydrate annotation mining is not the relevant interpretation layer
- The scorecard was run with default thresholds (e.g. fold-change 2-3, multiplication
  factor 1.5, p <= 0.05) rather than the Dataset 3 thresholds (fold-change 2 and 4,
  p <= 0.001), so the quadrant records will not match the reported signatures
- The goal is to preselect carbohydrate genes before analysis — the scorecard explicitly
  maintains a data-driven feature extraction approach that bypasses gene preselection
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: scorecard_functions
  role: Python library whose generate_parameters, data_loading, scorecard_legend,
    scorecard, and reconstruct_scorecard functions produce the quadrant records and
    reconstructed figure that are then annotated and validated for carbohydrate signatures
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV log2 fold-change and adjusted p-value table into memory as a
    Pandas DataFrame for scorecard processing
- name: Python
  role: Scripting environment in which the scorecard library is imported and the annotation
    mining of identified genes is performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:e07b5b1d1e7d2eece006c9c0c6ee2a807d2c120dd9adeeb76bafbe67ada1bb49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# carbohydrate-metabolism-gene-signature-interpretation

## Summary

Interprets scorecard outputs from Streptococcus mutans RNA-seq data across dietary carbohydrates (glucose, sucrose, lactose, xylitol) to identify PEP-PTS/carbohydrate-metabolism gene signatures, such as SMU_494 downregulation under sucrose and PTS-system genes (lacE, lacF, bglP, ptcA/ptcB/ptcC, celR) in regions of interest.

## When to use

Use when a reconstructed scorecard from S. mutans (SMU) transcriptomic comparisons under different dietary carbohydrates must be annotated/validated against known PEP-PTS and carbohydrate-metabolism gene families.

## When NOT to use

- Input data are not transcriptomic comparisons under different carbohydrate/dietary conditions (e.g. antibiotic exposure datasets 1-2 or metal-surface dataset 4), where PTS/carbohydrate annotation mining is not the relevant interpretation layer
- The scorecard was run with default thresholds (e.g. fold-change 2-3, multiplication factor 1.5, p <= 0.05) rather than the Dataset 3 thresholds (fold-change 2 and 4, p <= 0.001), so the quadrant records will not match the reported signatures
- The goal is to preselect carbohydrate genes before analysis — the scorecard explicitly maintains a data-driven feature extraction approach that bypasses gene preselection

## Inputs

- Preprocessed CSV table of log2 fold-change and adjusted p-values for S. mutans (SMU) genes under glucose, sucrose, lactose, xylitol conditions, with gene-name column
- Quadrant output records from scorecard_functions.scorecard(df,param_dict) run with fold-change thresholds 2 and 4 (multiplication factor 2) and p <= 0.001
- Reconstructed scorecard figure from scorecard_functions.reconstruct_scorecard(df,param_dict)

## Outputs

- Annotated list of PEP-PTS/carbohydrate-metabolism gene signatures (lacE, lacF, SMU_311, SMU_312, bglP, SMU_313, ptcA, ptcB, ptcC, celR operon, SMU_314, bgl, pttB)
- Validation record confirming SMU_494, pdhC and treA are consistently downregulated under sucrose
- Interpreted carbohydrate scorecard summary linking gene expression signatures to dietary carbohydrate conditions

## How to apply

After running the scorecard workflow on Dataset 3 with fold-change thresholds of 2 and 4 (multiplication factor 2) and significance level p <= 0.001, examine the quadrant output records of the reconstructed scorecard for carbohydrate-related gene symbols. Perform annotation mining on the identified genes, looking for recurring keywords such as the PTS (phosphotransferase) system (lacE, lacF, SMU_311, SMU_312, bglP, SMU_313, ptcA, ptcB, ptcC), the cel (celR) operon, and other PEP-PTS-related entries (SMU_313/SMU_314, bgl, pttB). Validate the interpretation by confirming that SMU_494, pdhC and treA are consistently downregulated under sucrose (SMU_494 is the reported highly bioactive gene always downregulated for Suc) and that the listed PEP-PTS genes appear in the regions-of-interest records. Acceptance criterion: the reconstructed scorecard exists as a rendered figure and SMU_494 is flagged as downregulated for sucrose, with the PEP-PTS/carbohydrate genes present in the quadrant output records.

## Related tools

- **scorecard_functions** (Python library whose generate_parameters, data_loading, scorecard_legend, scorecard, and reconstruct_scorecard functions produce the quadrant records and reconstructed figure that are then annotated and validated for carbohydrate signatures) — github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV log2 fold-change and adjusted p-value table into memory as a Pandas DataFrame for scorecard processing)
- **Python** (Scripting environment in which the scorecard library is imported and the annotation mining of identified genes is performed)

## Examples

```
param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- The reconstructed scorecard exists as a rendered figure (via scorecard_functions.reconstruct_scorecard with fig_size/use_figsize and figsize_factor controls)
- SMU_494 is flagged as downregulated for sucrose in the quadrant output records, matching the reported finding that it is 'always downregulated for Suc'
- The listed PEP-PTS/carbohydrate genes (SMU_313/SMU_314, ptcA/ptcB/ptcC, bgl, celR, pttB, lacE, lacF, bglP) appear in the quadrant output records for the carbohydrate conditions
- Dataset 3 thresholds were actually used: fold-change thresholds of 2 and 4 (multiplication factor 2) and significance level p <= 0.001
- Annotation mining shows PTS (phosphotransferase) system keywords and the cel (celR) operon among the most frequent keywords across Dataset 3 scorecards

## Limitations

- Dataset 3 thresholds were deliberately modified (fold-change 2 and 4, multiplication factor 2, p <= 0.001) to detect fewer outliers, so sensitivity to moderate expression changes is reduced
- Interpretation depends on annotation mining of gene symbols; the approach targets genes that emerge from data-driven scorecard regions of interest, so experience-driven preselection of carbohydrate genes would bypass the intended workflow
- Validation is figure- and record-based: the scorecard is accepted if it exists as a rendered figure and SMU_494 is flagged downregulated for sucrose; no independent biological assay confirmation is part of this skill

## Evidence

- [other] Annotation mining of the genes identified across Dataset 3 scorecards revealed that frequently appearing keywords were the PTS (phosphotransferase) system: [evidence span withheld — anchor status: not_found]
- [discussion] The scorecard analysis of dataset 3 revealed that a highly bioactive gene was SMU_494, always downregulated for Suc.: "The scorecard analysis of dataset 3, revealed that a highly bioactive gene was SMU_494, always downregulated for Suc."
- [results] the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p: "the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [intro] One of the goals of the scorecard is to bypass gene preselection, maintaining a data-driven feature extraction approach.: "One of the goals of the scorecard is to bypass gene preselection, maintaining a data-driven feature extraction approach."
