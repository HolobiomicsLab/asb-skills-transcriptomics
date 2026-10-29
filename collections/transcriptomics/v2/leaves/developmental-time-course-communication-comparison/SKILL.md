---
name: developmental-time-course-communication-comparison
description: Use when two or more scRNA-seq time points of a developing tissue are
  available with cell-type labels, and the goal is to identify stage-shared vs stage-specific
  communications and shifts in dominant communicating cell types (e.g., dermal-to-epidermal).
when_to_use_negative:
- The analysis targets a single time point or condition with no comparison stage —
  just run MDIC3 scoring; no cross-stage enumeration or community-center shift analysis
  is possible
- The biological question requires named ligand-receptor pairs as the primary output
  — MDIC3 is L-R independent and only supports a simple post-hoc L-R extraction against
  CellChatDB (lucky.readLRDB / lucky.LR_exp), so use CellChat, CellPhoneDB, or iTALK
  instead
- Cell labels are absent or unreliable for one stage, since the cell-type-level communication
  matrices and community centers depend on existing cell classification labels
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: MDIC3
  role: Unsupervised matrix-decomposition method (A = R*Sigma*W via GNIPLR GRN, SVD,
    and W=(R*Sigma)+A) that infers per-stage cell-cell communication matrices compared
    across time points
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network (GRN adjacency matrix R) for each stage's
    expression matrix via LASSO/linear projection and lagged regression
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChat
  role: Alternative L-R coexpression-based method and source of the CellChatDB ligand-receptor
    database used for optional post-hoc L-R pair extraction
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:76ce5b0f34b711fdab55255bdf89ca8e3450d5366102f3623de95ad9bd4501f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# developmental-time-course-communication-comparison

## Summary

Compare MDIC3-inferred cell-cell communication matrices across developmental stages (e.g., E13.5 vs E14.5 mouse skin from GSE122043) to enumerate shared and stage-specific communications and track shifts in the dominant signaling communities over time.

## When to use

Use when two or more scRNA-seq time points of a developing tissue are available with cell-type labels, and the goal is to identify stage-shared vs stage-specific communications and shifts in dominant communicating cell types (e.g., dermal-to-epidermal).

## When NOT to use

- The analysis targets a single time point or condition with no comparison stage — just run MDIC3 scoring; no cross-stage enumeration or community-center shift analysis is possible
- The biological question requires named ligand-receptor pairs as the primary output — MDIC3 is L-R independent and only supports a simple post-hoc L-R extraction against CellChatDB (lucky.readLRDB / lucky.LR_exp), so use CellChat, CellPhoneDB, or iTALK instead
- Cell labels are absent or unreliable for one stage, since the cell-type-level communication matrices and community centers depend on existing cell classification labels

## Inputs

- Per-time-point scRNA-seq expression matrices as gene-by-cell tab-delimited exp.txt (no header field in the first row; at least 10 cells recommended), e.g., E13.5 and E14.5 mouse skin from GEO GSE122043
- Per-time-point cell-label metadata files (metadata.txt: first column cell names matching exp.txt first row exactly, second column cell type labels)
- GNIPLR run parameters: number of work processes and block-calculation step size (number of genes or <=3000)

## Outputs

- Per-time-point celltype_communication.txt matrices (row i, column j = strength of signal from cell type i to cell type j)
- Per-time-point cellular_communication.txt single-cell communication matrices
- List of shared communications between stages (41 for E13.5 vs E14.5 mouse skin) and stage-specific communications (e.g., E14.5 bidirectional spinous-basal-P signaling)
- Top-10 communication-strength community rankings per stage with center cell types, showing the dermal-to-epidermal shift

## How to apply

For each time point, format the scRNA-seq data as a gene-by-cell exp.txt plus a cell-label metadata.txt (first metadata column must exactly match the expression file's first row), load them with lucky.readexp and lucky.readlabel, and infer the GRN with lucky.GRN_GNIPLR, setting the GNIPLR block-calculation step size to the number of genes or a value of 3000 or less and choosing the number of work processes per machine configuration. Run lucky.MDIC3_score, which factorizes the expression matrix A into R*Sigma*W (R is the GRN adjacency matrix, Sigma from singular value decomposition, W=(R*Sigma)+A), where the sign of Wij encodes communication direction and |Wij| the strength; save results with lucky.MDIC3_scoresave to get celltype_communication.txt and cellular_communication.txt. Then compare the per-stage cell-type communication matrices: enumerate communications present at both stages (the E13.5/E14.5 mouse skin comparison yields 41 shared communications), identify stage-specific ones such as the E14.5-specific bidirectional paracrine signaling between spinous and basal-P epidermal cells (Figure 4B), and rank the top-10 communication-strength communities per stage to check that their center cell types shift from dermal cells at E13.5 to epidermal cells at E14.5 (Figures 4C-4D).

## Related tools

- **MDIC3** (Unsupervised matrix-decomposition method (A = R*Sigma*W via GNIPLR GRN, SVD, and W=(R*Sigma)+A) that infers per-stage cell-cell communication matrices compared across time points) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network (GRN adjacency matrix R) for each stage's expression matrix via LASSO/linear projection and lagged regression) — https://github.com/zyllluck/GNIPLR
- **CellChat** (Alternative L-R coexpression-based method and source of the CellChatDB ligand-receptor database used for optional post-hoc L-R pair extraction) — https://github.com/sqjin/CellChat

## Examples

```
python MDIC3.py -exp=E14.5_exp.txt -label=E14.5_metadata.txt -grnchoose='GNIPLR' -process=8 -step=3000 -out=E14.5_results
```

## Evaluation signals

- The reproduced stage comparison matches published Figure 4B: 41 communications shared between E13.5 and E14.5 mouse skin, including the E14.5-specific bidirectional paracrine communication between spinous and basal-P cells
- Top-10 communication-strength community centers shift from dermal cells at E13.5 to epidermal cells at E14.5, matching Figures 4C-4D
- Both output files (celltype_communication.txt, cellular_communication.txt) are produced per stage with the documented layout: first row/column are cell labels; entry (i,j) = strength of signal sent by i and received by j
- Metadata.txt first column matches the exp.txt first row exactly for every time point, and each stage has at least 10 cells per analysis
- Signs of Wij entries are interpretable as communication direction and |Wij| as strength in the per-stage matrices, consistent with W=(R*Sigma)+A

## Limitations

- GRN inference quality directly affects MDIC3 communication results; the accuracy of the GRN (GNIPLR) is an open dependency
- When the number of cells exceeds the number of genes, some information may be missing in the right submatrix W
- The original single-cell expression matrix used as the middle submatrix may contain noise
- Communication networks of individual cells are difficult to verify experimentally, so validation is done at the cell-type level
- No software version numbers are stated for MDIC3, GNIPLR, or the benchmarked alternative methods

## Evidence

- [intro] the dominant communication signaling during the developmental stages from embryonic day 13.5 (E13.5) to E14.5 changes from dermal cells to epidermal cells: "the dominant communication signaling during the developmental stages from embryonic day 13.5 (E13.5) to E14.5 changes from dermal cells to epidermal cells"
- [discussion] MDIC3 factorizes the matrix A into three matrices R, Σ, and W: "MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW"
- [discussion] The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [methods] ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index): "ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index)"
- [readme] the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type: "More specifically, the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type."
