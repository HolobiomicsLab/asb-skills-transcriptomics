---
name: distance-correlation-statistics
description: Use when you have a gene-by-cell scRNA-seq matrix and a redPATH pseudotime
  ordering and need to rank/select genes whose expression correlates with the developmental
  trajectory, such as markers of transitions between cell states.
when_to_use_negative:
- When pseudotime results perform poorly — the article notes this approach 'may fail
  when the pseudotime results perform poorly', so validate redpath() output first
- When the input is already a filtered/subsetted gene matrix — dCor/MIC should be
  computed on the FULL expression matrix, not the GO-subsetted, dispersion-filtered
  matrix used for pseudotime inference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: redPATH
  role: Provides calc_correlation (type = "dcor" / "mic"), gene_selection, and gene_selection_intersection
    for gene discovery along pseudotime
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:28aa5f11f06233b827935b7da6199416e5608ad1bcbbb81363647f94c293bba9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - redPATH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# distance-correlation-statistics

## Summary

Applies redPATH's gene-discovery statistics — distance correlation (dCor) and the Maximal Information Coefficient (MIC) — between each gene's expression profile and the redPATH-inferred pseudotime to identify genes that change significantly along cell development, e.g. CSF1R in MGH45/MGH107 glioma data.

## When to use

Use when you have a gene-by-cell scRNA-seq matrix and a redPATH pseudotime ordering and need to rank/select genes whose expression correlates with the developmental trajectory, such as markers of transitions between cell states.

## When NOT to use

- When pseudotime results perform poorly — the article notes this approach 'may fail when the pseudotime results perform poorly', so validate redpath() output first
- When the input is already a filtered/subsetted gene matrix — dCor/MIC should be computed on the FULL expression matrix, not the GO-subsetted, dispersion-filtered matrix used for pseudotime inference

## Inputs

- Full gene-by-cell scRNA-seq expression matrix (m genes × n cells), e.g. llorens_exprs or mgh107_portal_exprs
- redPATH pseudotime ordering vector computed by redpath() on the GO-subsetted, dispersion-filtered matrix
- Numeric selection threshold (0.5 in README example; 0.65 for the NSC-Llorens proliferation/differentiation analysis)

## Outputs

- dcor_result: per-gene distance correlation scores against pseudotime
- mic_result: per-gene Maximal Information Coefficient scores against pseudotime
- selected_result: m-genes × n-cells matrix restricted to significant genes passing the dCor/MIC threshold

## How to apply

Load the full gene-by-cell expression matrix (e.g. llorens_exprs for the 145-cell neural stem cell example or mgh107_portal_exprs for the 252-cell MGH107 glioma example) and compute the redPATH pseudotime with redpath(). Call calc_correlation twice on the full expression matrix against redpath_pseudotime: once with type = "dcor" (distance correlation, energy package) and once with type = "mic" (Maximal Information Coefficient, minerva package). Then retain significant genes via gene_selection (union) or gene_selection_intersection with a threshold — 0.5 in the README example, 0.65 to recover Foxm1, Tubb5, and Nek2 on NSC-Llorens — which returns a selected genes-by-cells matrix. Downstream, gene scores can be cross-validated across datasets, e.g. CSF1R reporting dCor 0.95 / MIC 0.78 in MGH45 and dCor 0.93 / MIC 0.78 in MGH107.

## Related tools

- **redPATH** (Provides calc_correlation (type = "dcor" / "mic"), gene_selection, and gene_selection_intersection for gene discovery along pseudotime) — https://github.com/tinglabs/redPATH

## Examples

```
dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"); mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic"); selected_result <- gene_selection(input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)
```

## Evaluation signals

- CSF1R row in a combined MGH45/MGH107 score table must show dCor ≈ 0.95 and MIC ≈ 0.78 (MGH45) and dCor ≈ 0.93 and MIC ≈ 0.78 (MGH107), within small numerical tolerance
- Union selection at threshold = 0.65 on NSC-Llorens must recover Foxm1, Tubb5, and Nek2, with Dcx and Sp9 also among selected genes
- Every retained gene must have at least one score (dCor or MIC) above the selection threshold (0.5 or 0.65 as configured)
- selected_result must be a gene-by-cell matrix whose row count equals the number of genes passing the threshold

## Limitations

- The gene-discovery approach may fail when the pseudotime results perform poorly, since dCor/MIC are computed against the inferred ordering
- Threshold choice materially changes the retained gene set (0.5 in the README example vs 0.65 for the proliferation/differentiation NSC analysis)

## Evidence

- [readme] dcor and MIC calculation commands: "dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor")"
- [readme] gene selection function: "selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [discussion] gene discovery analysis in discussion: "2) gene discovery using dCor and MIC"
- [discussion] caveat about pseudotime quality: "this approach may fail when the pseudotime results perform poorly"
