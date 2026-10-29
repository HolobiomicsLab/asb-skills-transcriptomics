---
name: cross-condition-communication-comparison
description: Use when the same scRNA-seq communication inference (e.g., MDIC3) has
  been run on two matched conditions of a dataset (e.g., GEO GSE147424 LS and NL skin)
  and the goal is to distinguish condition-specific communications from shared ones.
when_to_use_negative:
- Only a single condition is available — with no matched second condition there is
  nothing to compare and per-condition MDIC3 analysis alone suffices
- The analysis goal is ligand-receptor pair identification within one condition (use
  the MDIC3 LR_exp / readLRDB workflow instead of cross-condition comparison)
- The two conditions use different cell-type label schemes or incomparable preprocessing,
  making heatmap cells non-corresponding
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Infers the per-condition cell-type and single-cell communication matrices
    that are compared across conditions
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the GRN adjacency matrix used as the left submatrix R by MDIC3 in each
    condition
  repo: https://github.com/zyllluck/GNIPLR
- name: pheatmap
  role: R script renders each celltype_communication.txt as a blue-white-red heatmap
    (no clustering) for the LS vs NL visual comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:70ae1f344fb0c42412fdbf01b5a03d22589a2278ded5cb8cedd1f6c0788ccfa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - GNIPLR
  - MDIC3
  - Python
  - pheatmap
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-condition-communication-comparison

## Summary

Jointly compare MDIC3-inferred cell-type communication matrices (celltype_communication.txt) across two biological conditions (e.g., atopic dermatitis lesional vs nonlesional skin) to identify both shared and condition-specific cell-cell communications, such as the LS-specific InflameFIB-to-InflameDC signal.

## When to use

Use when the same scRNA-seq communication inference (e.g., MDIC3) has been run on two matched conditions of a dataset (e.g., GEO GSE147424 LS and NL skin) and the goal is to distinguish condition-specific communications from shared ones.

## When NOT to use

- Only a single condition is available — with no matched second condition there is nothing to compare and per-condition MDIC3 analysis alone suffices
- The analysis goal is ligand-receptor pair identification within one condition (use the MDIC3 LR_exp / readLRDB workflow instead of cross-condition comparison)
- The two conditions use different cell-type label schemes or incomparable preprocessing, making heatmap cells non-corresponding

## Inputs

- celltype_communication.txt per condition (row i, column j = signal sent by cell type i, received by cell type j)
- cellular_communication.txt per condition (single-cell level communication matrix)
- MDIC3 outputs from two matched conditions (e.g., LS and NL skin from GEO GSE147424)
- pheatmap R visualization script (blue-white-red scale, no clustering)

## Outputs

- Side-by-side pheatmap plots (plot.pdf) of LS and NL cell-type communication matrices
- List of condition-specific communications (e.g., LS-only InflameFIB→InflameDC)
- Enumerated list of shared communications (e.g., 8 immune-cell and 14 inflammatory-cell communications present in both NL and LS skin)
- Joint comparison figure reproducing Figure 4E of the MDIC3 paper

## How to apply

Run lucky.MDIC3_score per condition (LS and NL) on GSE147424 expression matrices with GRNs inferred by lucky.GRN_GNIPLR (block step size ≤500, process pool sized to the machine), then save each result with lucky.MDIC3_scoresave to obtain celltype_communication.txt and cellular_communication.txt, where row i column j is the signal sent by cell type i and received by cell type j. Visualize each cell-type matrix with the provided pheatmap R script (blue-white-red scale, no clustering, output plot.pdf), where red squares mark significant communications. Place the LS and NL heatmaps side by side and jointly enumerate communications: verify which communications appear in both conditions (shared, e.g., the 8 immune-cell and 14 inflammatory-cell signals in NL and LS skin) and which are condition-specific (LS-only InflameFIB→InflameDC). The comparison is qualitative — accept when the rendered pattern agrees with the reported Figure 4E: InflameFIB→InflameDC significant (red) in LS only, and the shared-communication counts match 8 immune-cell and 14 inflammatory-cell communications. Shared signals in nonlesional skin (e.g., inflammatory-cell communications) can be interpreted as evidence of subclinical inflammation in NL skin.

## Related tools

- **MDIC3** (Infers the per-condition cell-type and single-cell communication matrices that are compared across conditions) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the GRN adjacency matrix used as the left submatrix R by MDIC3 in each condition) — https://github.com/zyllluck/GNIPLR
- **pheatmap** (R script renders each celltype_communication.txt as a blue-white-red heatmap (no clustering) for the LS vs NL visual comparison)

## Examples

```
python MDIC3.py -exp=LS_exp.txt -label=LS_metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=LS_out; python MDIC3.py -exp=NL_exp.txt -label=NL_metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=NL_out
```

## Evaluation signals

- The rendered LS heatmap marks the InflameFIB→InflameDC communication as significant (red) while the NL heatmap does not
- The count of immune-cell communications shared by both conditions equals 8
- The count of inflammatory-cell communications (inflame FIB, inflame DC, inflame TC) present in both NL and LS skin equals 14
- Each heatmap uses blue-white-red scale with no clustering and is saved as plot.pdf, with row/column cell labels matching the saved celltype_communication.txt matrices
- Shared communications in NL skin are interpretable as subclinical inflammation, consistent with the original atopic dermatitis study

## Limitations

- No quantitative threshold is reported for calling a communication significant in the joint comparison, so acceptance is qualitative agreement with the reported Figure 4E pattern
- Shared signals in nonlesional skin may reflect subclinical inflammation rather than true healthy-state communication, complicating interpretation
- MDIC3 results depend on GRN accuracy (GNIPLR or imported GRN), so cross-condition differences may reflect GRN inference noise rather than biology

## Evidence

- [other] MDIC3 identified an LS-specific communication sent from inflammatory FIB and received by inflammatory DC, consistent with the experimental results of the original study: [evidence span withheld — anchor status: not_found]
- [other] 14 communication signals related to inflammatory cells appeared in both NL and LS skin, suggesting subclinical inflammation in NL skin: [evidence span withheld — anchor status: not_found]
- [readme] the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type: "the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type"
- [discussion] the accuracy of the GRN can affect the results of cell-cell communication for MDIC3: "the accuracy of the GRN can affect the results of cell-cell communication for MDIC3."
