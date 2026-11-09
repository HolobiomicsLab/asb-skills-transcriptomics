---
name: maximal-information-coefficient-scoring
description: Use when gene-by-cell scRNA-seq expression matrices and a redPATH pseudotime
  ordering are available and you need to rank genes by non-linear association with
  developmental/cancer trajectory, e.g. recovering CSF1R or Foxm1/Tubb5/Nek2.
when_to_use_negative:
- When pseudotime results perform poorly — the article notes this gene-discovery approach
  may fail when the pseudotime results perform poorly
- When you only need linear associations and already have dCor scores sufficient for
  selection — MIC adds non-linear detection but at higher computational cost
- When the goal is cell cycle state assignment rather than gene discovery along pseudotime
  — use the reCAT-based G0-like cell detection or HMM segmentation functions instead
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: redPATH
  role: Provides calc_correlation(type='mic') to compute per-gene MIC against redPATH
    pseudotime, plus gene_selection / gene_selection_intersection for threshold-based
    retention of significant genes
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:f9a5746b20d937bc8a06f97f4cfa6bcd7036bd8606df9a35b7a9e04e878a6225
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - redPATH
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# maximal-information-coefficient-scoring

## Summary

Computes Maximal Information Coefficient (MIC) scores between each gene's expression profile and the redPATH-inferred pseudotime using calc_correlation(type='mic'), to discover genes whose non-linear dependence on developmental ordering is significant in single-cell RNA-seq datasets such as MGH45, MGH107, and the NSC-Llorens neural stem cell data.

## When to use

Use when gene-by-cell scRNA-seq expression matrices and a redPATH pseudotime ordering are available and you need to rank genes by non-linear association with developmental/cancer trajectory, e.g. recovering CSF1R or Foxm1/Tubb5/Nek2.

## When NOT to use

- When pseudotime results perform poorly — the article notes this gene-discovery approach may fail when the pseudotime results perform poorly
- When you only need linear associations and already have dCor scores sufficient for selection — MIC adds non-linear detection but at higher computational cost
- When the goal is cell cycle state assignment rather than gene discovery along pseudotime — use the reCAT-based G0-like cell detection or HMM segmentation functions instead

## Inputs

- Full gene-by-cell expression matrix (m genes x n cells), e.g. llorens_exprs (145 cells) or mgh107_portal_exprs (MGH107, 252 cells)
- redPATH pseudotime ordering from redpath() on the filtered/GO-subsetted matrix
- Optional: cell type labels (e.g. llorens_labels) for downstream plotting of selected genes

## Outputs

- Per-gene MIC score vector from calc_correlation(type = 'mic')
- Selected gene-by-cell expression matrix with genes passing the MIC/dCor threshold (from gene_selection or gene_selection_intersection)
- Combined dCor/MIC score table (e.g. gene_score_table.csv with dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107 columns, or selected_genes_dcor_mic.csv)

## How to apply

Load the full gene-by-cell expression matrix (e.g. mgh107_portal_exprs for MGH107's 252 cells, or llorens_exprs for the 145-cell neural stem cell data) and compute the redPATH pseudotime via the redpath() function, which can be applied after GO-subsetting (get_GO_exp) and dispersion filtering (filter_exp, dispersion_threshold = 10). Run calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = 'mic') to obtain a per-gene MIC vector using the minerva implementation. Combine MIC scores with dCor scores via gene_selection (union, threshold = 0.5) or gene_selection_intersection; for the NSC-Llorens proliferation/differentiation analysis a higher threshold = 0.65 is used. Validate against reported values: CSF1R has MIC 0.78 in both MGH45 and MGH107, and Foxm1, Tubb5, Nek2 (plus markers Dcx and Sp9) should be retained at the 0.65 threshold.

## Related tools

- **redPATH** (Provides calc_correlation(type='mic') to compute per-gene MIC against redPATH pseudotime, plus gene_selection / gene_selection_intersection for threshold-based retention of significant genes) — https://github.com/tinglabs/redPATH

## Examples

```
mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic"); selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.65)
```

## Evaluation signals

- CSF1R row in the MGH45/MGH107 score table has MIC equal to 0.78 in both datasets (within small numerical tolerance), matching dCor values of 0.95 (MGH45) and 0.93 (MGH107)
- Applying gene_selection with threshold = 0.65 on NSC-Llorens recovers Foxm1, Tubb5, and Nek2, with marker genes Dcx and Sp9 among selected genes
- Every retained gene has a dCor or MIC score above the selection threshold (0.5 for union/intersection default, 0.65 for the NSC analysis)
- Output table contains one row per gene with the expected columns (e.g. dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107) and no missing gene identifiers
- gene_selection returns a m genes by n cells matrix containing only the selected significant genes

## Limitations

- This gene discovery approach may fail when the pseudotime results perform poorly, since MIC is computed against the inferred ordering
- Removing cycling genes to remove cell cycle effects, as done in traditional trajectory inference analyses, is deliberately avoided by redPATH — MIC may therefore reflect proliferation-coupled signals (e.g. Foxm1, Tubb5, Nek2) that confound differentiation-only interpretation
- The union vs intersection selection choice and the threshold (0.5 vs 0.65) materially change the retained gene set; the reported NSC result requires union selection at 0.65

## Evidence

- [methods] calc_correlation with type='mic' usage: "dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor")"
- [readme] gene selection thresholds: "selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [results] CSF1R dCor/MIC reported values: [evidence span withheld — anchor status: not_found]
- [discussion] NSC gene discovery finding: "2) gene discovery using dCor and MIC"
- [discussion] pseudotime dependence caveat: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
- [methods] MGH107 example dataset availability: "a diseased example (MGH107 - 252 cells) dataset is also provided from 2017 (2)"
