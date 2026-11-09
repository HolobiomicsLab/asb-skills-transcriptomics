---
name: gene-significance-selection-thresholding
description: Use when you have a gene-by-cell expression matrix and a redPATH pseudotime
  ordering and need to rank or filter genes for significance along the trajectory
  using dCor and MIC statistics.
when_to_use_negative:
- When no pseudotime ordering has been inferred yet — correlations require redpath_pseudotime
  and 'this approach may fail when the pseudotime results perform poorly'.
- When the input is already a filtered/GO-subset matrix (filtered_GO_data) intended
  for pseudotime inference, not the full expression matrix that calc_correlation expects.
- When the goal is cell-state segmentation (ON/OFF HMM states) rather than gene significance
  ranking — use calc_all_hmm_diff_parallel instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Provides calc_correlation (type = "dcor" / "mic"), gene_selection, and gene_selection_intersection
    used to score and threshold genes against pseudotime
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:ede6b78a7c2e662628428e75a72d345eaec2b617483d2c53dc33f35be74d47b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - redPATH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gene-significance-selection-thresholding

## Summary

redPATH's gene-discovery statistics: compute distance correlation (dCor) and maximal information coefficient (MIC) of each gene's expression against the inferred pseudotime, then retain significant genes via union or intersection selection at threshold 0.5. This is the mechanism by which the study surfaced CSF1R (dCor 0.95/MIC 0.78 in MGH45; dCor 0.93/MIC 0.78 in MGH107) at the microglia/macrophage-to-malignant transition.

## When to use

Use when you have a gene-by-cell expression matrix and a redPATH pseudotime ordering and need to rank or filter genes for significance along the trajectory using dCor and MIC statistics.

## When NOT to use

- When no pseudotime ordering has been inferred yet — correlations require redpath_pseudotime and 'this approach may fail when the pseudotime results perform poorly'.
- When the input is already a filtered/GO-subset matrix (filtered_GO_data) intended for pseudotime inference, not the full expression matrix that calc_correlation expects.
- When the goal is cell-state segmentation (ON/OFF HMM states) rather than gene significance ranking — use calc_all_hmm_diff_parallel instead.

## Inputs

- Full gene-by-cell expression matrix (m genes rows x n cells cols), e.g. input_data or mgh107_portal_exprs (MGH107, 252 cells)
- redpath_pseudotime vector from the redpath() function for the same cells

## Outputs

- dcor_result: per-gene distance correlation scores
- mic_result: per-gene maximal information coefficient scores
- selected_result: m genes by n cells matrix of significant genes (union or intersection)
- Ranked per-gene table of dCor/MIC values across datasets (e.g. gene_score_table.csv with dcor_mgh45, mic_mgh45, dcor_mgh107, mic_mgh107 columns)

## How to apply

Run calc_correlation twice on the full expression matrix against redpath_pseudotime, once with type = "dcor" (distance correlation via the energy package) and once with type = "mic" (maximal information coefficient via the minerva package). Then call gene_selection for union selection or gene_selection_intersection for intersection selection, passing both results with threshold = 0.5 to retain genes significant under both statistics. The output selected_result is an m-genes by n-cells matrix of the selected significant genes, which can be fed to downstream steps such as calc_all_hmm_diff_parallel or compiled into a ranked table of dCor/MIC values per gene. Validate by checking that known transition markers reproduce the published values, e.g. CSF1R should show dCor 0.95 and MIC 0.78 in MGH45 and dCor 0.93 and MIC 0.78 in MGH107, with numerous overlaps between the two datasets.

## Related tools

- **redPATH** (Provides calc_correlation (type = "dcor" / "mic"), gene_selection, and gene_selection_intersection used to score and threshold genes against pseudotime) — https://github.com/tinglabs/redPATH

## Examples

```
dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"); mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic"); selected_result <- gene_selection(input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)
```

## Evaluation signals

- CSF1R row must contain dCor 0.95 and MIC 0.78 for MGH45 and dCor 0.93 and MIC 0.78 for MGH107 (within numerical tolerance).
- gene_selection with threshold = 0.5 must return a matrix with one row per gene passing the threshold and the original n cells as columns.
- The union (gene_selection) result should be a superset of the intersection (gene_selection_intersection) result at the same threshold.
- Top-ranked genes should overlap between the MGH45 and MGH107 analyses, as the study reports numerous overlaps between the two datasets.
- dcor and mic result vectors must cover the same gene identifiers as the full expression matrix rows.

## Limitations

- Scores depend on pseudotime quality; the article notes the approach may fail when the pseudotime results perform poorly.
- The 0.5 threshold is a fixed default in the README workflow; no per-dataset calibration is described.

## Evidence

- [methods] dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"): "dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor")"
- [readme] gene_selection threshold: "selected_result <- gene_selection_intersection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [discussion] gene discovery using dCor and MIC: "2) gene discovery using dCor and MIC"
- [abstract] CSF1R for macrophage reprogramming: "CSF1R for reprogramming macrophages to control tumor growth"
