---
name: union-threshold-gene-selection
description: Use when after computing dCor and MIC correlations of every gene's expression
  profile against redPATH pseudotime, you must retain genes significantly linked to
  proliferation/differentiation (e.g. recover Foxm1, Tubb5, Nek2 in the Llorens-Bobadilla
  NSC dataset).
when_to_use_negative:
- When the goal is only genes where BOTH statistics pass the cutoff — use gene_selection_intersection
  instead of union selection
- 'When pseudotime results perform poorly: the article notes this gene-discovery approach
  ''may fail when the pseudotime results perform poorly'''
- When input is not paired with a valid pseudotime ordering or correlation vectors
  (the function requires full_exprs_matrix, redpath_pseudotime, dcor_result, and mic_result
  together)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Provides the gene_selection (union), gene_selection_intersection, and calc_correlation
    (dcor/mic) functions that implement this selection step, plus llorens_exprs example
    data
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:421f0bb537870207e0b81202e4ee4ec83dd9eb52edeb3db18bc0eeb2be394072
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

# union-threshold-gene-selection

## Summary

Selects genes significantly associated with redPATH pseudotime by taking the union of genes whose distance correlation (dCor) or Maximal Information Coefficient (MIC) with the pseudotime exceeds a numeric threshold, producing a reduced gene-by-cell matrix of significant markers.

## When to use

Use when, after computing dCor and MIC correlations of every gene's expression profile against redPATH pseudotime, you must retain genes significantly linked to proliferation/differentiation (e.g. recover Foxm1, Tubb5, Nek2 in the Llorens-Bobadilla NSC dataset).

## When NOT to use

- When the goal is only genes where BOTH statistics pass the cutoff — use gene_selection_intersection instead of union selection
- When pseudotime results perform poorly: the article notes this gene-discovery approach 'may fail when the pseudotime results perform poorly'
- When input is not paired with a valid pseudotime ordering or correlation vectors (the function requires full_exprs_matrix, redpath_pseudotime, dcor_result, and mic_result together)

## Inputs

- m genes x n cells full expression matrix (e.g. llorens_exprs from the redPATH package)
- redpath_pseudotime: precomputed redPATH pseudo developmental time vector per cell
- dcor_result: per-gene distance correlation vector from calc_correlation(type='dcor')
- mic_result: per-gene Maximal Information Coefficient vector from calc_correlation(type='mic')
- threshold: numeric cutoff applied to dCor or MIC scores (0.5 in README example; 0.65 in the NSC-Llorens Foxm1/Tubb5/Nek2 reproduction)

## Outputs

- selected_result: gene-by-cell matrix containing only the selected significant genes (per README: 'a m genes by n cells matrix with the selected significant genes')
- downstream artifacts such as selected_genes_dcor_mic.csv listing retained genes and their dCor/MIC scores

## How to apply

Run calc_correlation twice on the full expression matrix (e.g. llorens_exprs, 145 cells) with type='dcor' and type='mic' against the redpath_pseudotime vector. Pass both correlation vectors plus the expression matrix and pseudotime into gene_selection, which implements union selection: a gene is kept if either its dCor or its MIC score exceeds the threshold (the README default is 0.5; the NSC-Llorens reproduction of Foxm1/Tubb5/Nek2 discovery uses threshold = 0.65). This differs from gene_selection_intersection, which requires both statistics to pass the cutoff and is the more conservative alternative. The function returns a gene-by-cell matrix restricted to the selected significant genes; compare the retained gene list against expected markers (proliferation/differentiation-coupled genes and known markers like Dcx and Sp9) to confirm correct application.

## Related tools

- **redPATH** (Provides the gene_selection (union), gene_selection_intersection, and calc_correlation (dcor/mic) functions that implement this selection step, plus llorens_exprs example data) — https://github.com/tinglabs/redPATH

## Examples

```
dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"); mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic"); selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.65)
```

## Evaluation signals

- Every retained gene has a dCor score or MIC score above the threshold (0.5 default; 0.65 in the NSC-Llorens task)
- The selected_result matrix has n cells (columns) matching the input expression matrix, with rows restricted to selected genes
- On the NSC-Llorens pseudotime, the retained set contains Foxm1, Tubb5, and Nek2 (proliferation/differentiation-coupled) plus marker genes Dcx and Sp9, matching the published analysis
- Re-running with gene_selection_intersection yields an equal or smaller gene set than union selection at the same threshold
- Varying the threshold downward monotonically increases (or leaves unchanged) the number of retained genes

## Limitations

- Discovery quality depends entirely on pseudotime quality: 'this approach may fail when the pseudotime results perform poorly'
- Union selection is permissive — a gene passing only one of dCor or MIC at the threshold is retained, so it can be noisier than intersection selection
- Correlating against pseudotime assumes a (near-)linear trajectory; the underlying coupling mechanism between proliferation and differentiation 'remains obscure and requires more integrative computational models'

## Evidence

- [methods] Union selection code line: "selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [readme] selected_result returns a m genes by n cells matrix with the selected significant genes: "selected_result returns a m genes by n cells matrix with the selected significant genes"
- [discussion] Gene discovery using dCor and MIC: "2) gene discovery using dCor and MIC"
- [readme] calc_correlation usage: [evidence span withheld — anchor status: external_doc]
- [discussion] Failure-mode caveat: "this approach may fail when the pseudotime results perform poorly"
