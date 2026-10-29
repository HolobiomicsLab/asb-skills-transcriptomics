---
name: multiple-testing-fdr-correction
description: Use when a group-comparison task yields one statistical test per feature
  (e.g., one per cell type in a DCNet-inferred TCGA abundance matrix) and per-feature
  p-values must be adjusted for multiple testing before filtering by effect size and
  q-value.
when_to_use_negative:
- The input is already a differential abundance/significance table with q-values —
  re-running correction would double-adjust the same tests.
- The comparison involves only a single hypothesis or a small pre-specified set of
  cell types where a family-wise/multiple-testing correction is unnecessary.
- The abundance matrix was produced by a different deconvolution method (CIBERSORTx,
  EPIC, MCP-counter, quanTIseq, TIMER, xCell) whose per-cell-type distributions differ;
  thresholds (FC >2 / <0.5, q<0.01) were calibrated on DCNet-inferred TCGA landscapes.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_2648
tools:
- name: Python
  role: Implements the per-cell-type fold-change computation, statistical tests, and
    p-value adjustment; the DCNet model itself was coded using the mxnet framework
    in Python
- name: DCNet
  role: Produces the upstream 434-cell-type abundance landscape from bulk RNA-seq
    that the differential tests are run on
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:65167f6b54956f905a75e67e814c3697485fb5b0780ab07329d05f672d5f6a4d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multiple-testing-fdr-correction

## Summary

Statistical workflow for testing per-cell-type abundance differences between two tumor groups (TCGA LUAD vs LUSC) on DCNet-inferred cell landscapes, adjusting per-cell-type p-values for multiple testing into q-values and filtering by fold change and significance threshold.

## When to use

Use when a group-comparison task yields one statistical test per feature (e.g., one per cell type in a DCNet-inferred TCGA abundance matrix) and per-feature p-values must be adjusted for multiple testing before filtering by effect size and q-value.

## When NOT to use

- The input is already a differential abundance/significance table with q-values — re-running correction would double-adjust the same tests.
- The comparison involves only a single hypothesis or a small pre-specified set of cell types where a family-wise/multiple-testing correction is unnecessary.
- The abundance matrix was produced by a different deconvolution method (CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) whose per-cell-type distributions differ; thresholds (FC >2 / <0.5, q<0.01) were calibrated on DCNet-inferred TCGA landscapes.

## Inputs

- DCNet-inferred cell-type abundance matrix for TCGA tumor samples (rows/cells = cell types, columns = samples)
- Sample subtype label table distinguishing TCGA LUAD from LUSC tumor samples
- Choice of statistical test for per-cell-type group comparison (run in Python)

## Outputs

- Per-cell-type table with fold change, raw p-value, and multiple-testing-adjusted q-value
- Filtered differential cell table (e.g., differential_cells.csv) restricted to fold change >2 or <0.5 and q<0.01, with directionality retained

## How to apply

Load the DCNet-inferred cell abundance matrix for the TCGA LUAD and LUSC tumor samples together with sample subtype labels, then compute the per-cell-type fold change in abundance between the two groups in Python. Run one statistical test of abundance difference per cell type between the subtypes, and adjust the resulting p-values for multiple testing to obtain q-values — since >400 cell types are tested at once (the DCNet landscape spans 434 cell types), uncorrected p-values would be invalid. Apply the joint filter of fold change >2 or <0.5 with q<0.01, keeping directionality of each cell type's change (higher in LUSC vs higher in LUAD). Write the filtered table with cell type, fold change, and q-value columns to a file such as differential_cells.csv. In the reference analysis this surfaced 15 differentially abundant cells in LUSC and 18 in LUAD with 12 common differential cell types.

## Related tools

- **Python** (Implements the per-cell-type fold-change computation, statistical tests, and p-value adjustment; the DCNet model itself was coded using the mxnet framework in Python)
- **DCNet** (Produces the upstream 434-cell-type abundance landscape from bulk RNA-seq that the differential tests are run on) — https://github.com/xindd/DCNet

## Evaluation signals

- All output q-values lie in [0,1] and every reported q-value equals or exceeds its raw p-value (adjustment must never decrease significance).
- Directionality is consistent: cell types flagged 'higher in LUSC' (Neuroepithelial, Myofibroblast, Basal) and 'higher in LUAD' (Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated) match the sign of their fold changes.
- The output table contains exactly the columns cell type, fold change, and q-value, and every retained row passes both thresholds (fold change >2 or <0.5 AND q<0.01).
- Count check against the reference result: 15 differentially abundant cell types in LUSC, 18 in LUAD, with 12 common differential cell types.
- Every one of the 434 cell types tested has both a p-value and a q-value (no cell type silently dropped before correction, which would bias the adjustment).

## Limitations

- The article does not report software version numbers or the specific p-value adjustment method, so exact q-values may not be reproducible without matching the implementation.
- Downstream results depend on DCNet-inferred abundances; no random seed or reproducibility controls for DCNet model training are mentioned, and the final dropout setting (0.1/0.3/0.5) used is not stated.
- DCNet marker-gene preprocessing discards cell types with no detected marker genes (33 discarded), so the tested family of cell types is smaller than the full 434-cell-type landscape.

## Evidence

- [methods] Perform a statistical test of abundance differences per cell type between the two subtypes and adjust p-values for multiple testing to obtain q-values.: [evidence span withheld — anchor status: not_found]
- [methods] Filter cell types meeting fold change >2 or <0.5 with q<0.01, retaining directionality: [evidence span withheld — anchor status: not_found]
- [results] There are 15 differentially expressed cells in LUSC and 18 in LUAD with 12 common differential cell types: [evidence span withheld — anchor status: not_found]
