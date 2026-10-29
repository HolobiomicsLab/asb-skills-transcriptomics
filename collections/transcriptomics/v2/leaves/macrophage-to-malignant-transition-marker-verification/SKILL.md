---
name: macrophage-to-malignant-transition-marker-verification
description: Use when verifying transition-marker genes (e.g. CSF1R) between microglia/macrophage
  and malignant cell populations in glioma scRNA-seq using redPATH dCor/MIC statistics
  along pseudotime.
when_to_use_negative:
- When the goal is branched-trajectory or HMM gene-state segmentation rather than
  pseudotime-correlation gene scoring — those require calc_all_hmm_diff_parallel /
  get_gene_cluster, not calc_correlation
- When pseudotime results perform poorly for the dataset — the article notes this
  approach may fail in that case
- When cell type labels already define the macrophage/malignant boundary and differential
  expression suffices — the dCor/MIC approach is designed for label-free gene discovery
  along pseudotime
edam_operation: http://edamontology.org/operation_0566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: redPATH
  role: R package providing redpath() for pseudotime inference and calc_correlation
    (type = "dcor" and "mic"), gene_selection, and gene_selection_intersection for
    gene-discovery scoring along pseudotime
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:d707b42dc2c8133166935a77802a3a388aadd216b59d0767ed3a5bd87da1711d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - redPATH
  edam_operation: http://edamontology.org/operation_0566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# macrophage-to-malignant-transition-marker-verification

## Summary

Reproduce the redPATH gene-discovery statistic pipeline (distance correlation and maximal information coefficient along inferred pseudotime) on the malignant glioma datasets MGH45 and MGH107 to verify that CSF1R — the marker of the microglia/macrophage-to-malignant-cell transition — scores dCor 0.95 / MIC 0.78 in MGH45 and dCor 0.93 / MIC 0.78 in MGH107.

## When to use

Use when verifying transition-marker genes (e.g. CSF1R) between microglia/macrophage and malignant cell populations in glioma scRNA-seq using redPATH dCor/MIC statistics along pseudotime.

## When NOT to use

- When the goal is branched-trajectory or HMM gene-state segmentation rather than pseudotime-correlation gene scoring — those require calc_all_hmm_diff_parallel / get_gene_cluster, not calc_correlation
- When pseudotime results perform poorly for the dataset — the article notes this approach may fail in that case
- When cell type labels already define the macrophage/malignant boundary and differential expression suffices — the dCor/MIC approach is designed for label-free gene discovery along pseudotime

## Inputs

- Gene-by-cell expression matrix for MGH45 malignant glioma dataset (m genes rows x n cells cols)
- Gene-by-cell expression matrix for MGH107 (Venteicher 2017 IDH-mutant glioma dataset, 252 cells; shipped with redPATH as mgh107_portal_exprs)
- redPATH pseudotime ordering for each dataset, from the redpath() function
- Target gene to verify: CSF1R

## Outputs

- Per-gene dCor and MIC score vectors for MGH45 (from calc_correlation type = "dcor" / "mic")
- Per-gene dCor and MIC score vectors for MGH107
- Combined ranked table gene_score_table.csv with columns dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107 (one row per gene)
- Verified CSF1R scores: dCor 0.95 / MIC 0.78 (MGH45) and dCor 0.93 / MIC 0.78 (MGH107)

## How to apply

Load the full gene-by-cell expression matrices for MGH45 and MGH107 (the MGH107 - 252 cell example ships with redPATH as mgh107_portal_exprs) and infer pseudotime with the redpath() function. Run calc_correlation with type = "dcor" and type = "mic" on each full expression matrix against its pseudotime to obtain per-gene distance correlation and maximal information coefficient scores. Optionally restrict to significant genes with gene_selection (union) or gene_selection_intersection at threshold = 0.5. Build a combined ranked table with one row per gene and columns dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107, then check the CSF1R row against the published values (MGH45: 0.95/0.78; MGH107: 0.93/0.78) within a small numerical tolerance, and save as gene_score_table.csv. The rationale is that CSF1R was identified in the paper as showing distinct change between microglia/macrophages and malignant cells, making it the anchor for cross-dataset validation with 'numerous overlaps between the two datasets'.

## Related tools

- **redPATH** (R package providing redpath() for pseudotime inference and calc_correlation (type = "dcor" and "mic"), gene_selection, and gene_selection_intersection for gene-discovery scoring along pseudotime) — https://github.com/tinglabs/redPATH

## Examples

```
dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"); mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic")
```

## Evaluation signals

- CSF1R row of gene_score_table.csv contains dCor/MIC ≈ 0.95/0.78 for MGH45 and ≈ 0.93/0.78 for MGH107 (within small numerical tolerance)
- gene_score_table.csv contains one row per gene with exactly the columns dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107
- gene_selection / gene_selection_intersection applied with threshold = 0.5 retains CSF1R among significant genes in both datasets
- Significant gene sets from MGH45 and MGH107 show numerous overlaps, consistent with the reported cross-dataset concordance
- Pseudotime inputs to calc_correlation are the redpath() orderings computed on each dataset's own full expression matrix

## Limitations

- This approach may fail when the pseudotime results perform poorly, since dCor/MIC are computed against the inferred pseudotime
- dCor and MIC calculations on full gene-by-cell matrices are computationally expensive (energy and minerva backends) for large gene counts
- The approach requires a linear pseudotime ordering from redpath(); it is not designed for strongly branched trajectories

## Evidence

- [abstract] CSF1R identified for reprogramming macrophages to control tumor growth: "CSF1R for reprogramming macrophages to control tumor growth"
- [methods] dcor calc_correlation call: "dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor")"
- [methods] gene_selection with threshold: "selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [readme] MGH107 example dataset: "a diseased example (MGH107 - 252 cells) dataset is also provided from 2017 (2)"
- [discussion] gene discovery using dCor and MIC as analysis function: "2) gene discovery using dCor and MIC"
