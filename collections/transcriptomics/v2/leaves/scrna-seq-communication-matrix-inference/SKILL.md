---
name: scrna-seq-communication-matrix-inference
description: Use when scRNA-seq data (expression matrix plus cell-type labels) must
  be analyzed for intercellular communication, especially for non-human species or
  when results should not be constrained by known L-R databases.
when_to_use_negative:
- When the analysis goal requires strictly literature-curated L-R coexpression-based
  inference in humans, where tools like CellChat, CellPhoneDB, or iTALK are the established
  alternatives
- When each cell type contains fewer than ~10 cells, as the README warns users to
  input single-cell gene expression data containing at least 10 cells
- When a time-course/time-series design is required for direct regulation inference
  — Equation 2's approach is limited by the requirement for time-series data, which
  MDIC3 circumvents but does not replace
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: 'Core algorithm: two-sided self-representation matrix decomposition (A=RΣW)
    of the expression matrix with the GRN as left submatrix to infer cell-cell communication;
    available as a Python package or MDIC3.py/MDIC3_LR.py scripts'
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Gene regulatory network inference (projection and lagged regression) producing
    the GRN adjacency matrix used as the left submatrix R in MDIC3; run with block
    calculation (-step ≤ 3000) and a multiprocessing process pool
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChat
  role: Source of the CellChatDB human/mouse L-R database used by MDIC3_LR.py to extract
    ligand-receptor pairs supporting the inferred communications
  repo: https://github.com/sqjin/CellChat
- name: GENIE3 / GRNBoost
  role: Alternative GRN inference tools whose adjacency matrices can replace the GNIPLR
    GRN via the -grnchoose='other' path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:4ce49151eaba943a871af81e85eb831f28cb0283733c12b9aafccf8645fbe031
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - GENIE3 / GRNBoost
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-seq-communication-matrix-inference

## Summary

Infer cell-cell communication from scRNA-seq data with MDIC3, which factorizes the gene-by-cell expression matrix via a gene regulatory network (GNIPLR) and two-sided matrix decomposition (R, Σ, W), yielding sender-receiver communication strengths independent of predefined ligand-receptor pairs and applicable to any species.

## When to use

Use when scRNA-seq data (expression matrix plus cell-type labels) must be analyzed for intercellular communication, especially for non-human species or when results should not be constrained by known L-R databases.

## When NOT to use

- When the analysis goal requires strictly literature-curated L-R coexpression-based inference in humans, where tools like CellChat, CellPhoneDB, or iTALK are the established alternatives
- When each cell type contains fewer than ~10 cells, as the README warns users to input single-cell gene expression data containing at least 10 cells
- When a time-course/time-series design is required for direct regulation inference — Equation 2's approach is limited by the requirement for time-series data, which MDIC3 circumvents but does not replace

## Inputs

- Single-cell gene expression *.txt file (genes as rows, cells as columns, cell names in first row, no header field)
- scRNA-seq metadata *.txt file (first column = cells, second column = cell-type labels, matching the expression file's first row)
- Optional GRN adjacency matrix *.txt file when bypassing GNIPLR (-grnchoose='other')
- Optional CellChatDB-derived Human_LR.txt ligand-receptor database for L-R extraction

## Outputs

- celltype_communication.txt (rows = sender cell types, columns = receiver cell types, values = communication strength)
- cellular_communication.txt (communication strength among individual cells)
- Filtered L-R pair list for target sender-receiver pairs with Bonferroni-corrected p values
- Heatmap (plot.pdf) of cell-type communications via the pheatmap R script

## How to apply

Format two inputs: a gene-by-cell expression *.txt file (genes as rows, cells as columns, first row contains only cell names, no header field) and a metadata *.txt pairing each cell with its cell-type label (first column must match the expression file's first row exactly; each cell type should contain at least 10 cells). Load data with lucky.readexp and lucky.readlabel, then infer the GRN with lucky.GRN_GNIPLR using block calculation with a '-step' parameter of 3000 or less and a user-selected '-process' count for the multiprocessing pool (a '-grnchoose=other' path accepts a GRN adjacency matrix from tools like GENIE3 or GRNBoost). Run lucky.MDIC3_score and save with lucky.MDIC3_scoresave: MDIC3 factorizes A into R (GRN adjacency), Σ (singular value decomposition), and W solved as W=(RΣ)+A, where the sign of Wij gives communication direction and |Wij| the strength. Optionally extract L-R pairs with MDIC3_LR.py against the CellChatDB-derived Human_LR.txt database, keeping pairs with Bonferroni-corrected p < 0.01, and visualize 'celltype_communication.txt' with a pheatmap R script (no clustering, blue-white-red ramp).

## Related tools

- **MDIC3** (Core algorithm: two-sided self-representation matrix decomposition (A=RΣW) of the expression matrix with the GRN as left submatrix to infer cell-cell communication; available as a Python package or MDIC3.py/MDIC3_LR.py scripts) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Gene regulatory network inference (projection and lagged regression) producing the GRN adjacency matrix used as the left submatrix R in MDIC3; run with block calculation (-step ≤ 3000) and a multiprocessing process pool) — https://github.com/zyllluck/GNIPLR
- **CellChat** (Source of the CellChatDB human/mouse L-R database used by MDIC3_LR.py to extract ligand-receptor pairs supporting the inferred communications) — https://github.com/sqjin/CellChat
- **GENIE3 / GRNBoost** (Alternative GRN inference tools whose adjacency matrices can replace the GNIPLR GRN via the -grnchoose='other' path)

## Examples

```
python MDIC3.py -exp=scRNA_expression_file -label=cell_label_file -grnchoose='GNIPLR' -process=process_value -step=GRN_calculation_step -out=results_output_fold
```

## Evaluation signals

- celltype_communication.txt is a symmetric-schematic square matrix whose first row and first column are cell labels; entry (i,j) is the strength of signal sent by cell type i and received by cell type j
- Input-format invariant: the first column of metadata.txt matches the first row of exp.txt cell names exactly
- L-R validation: extracted L-R list for human lupus nephritis contains CCL2-CCR2, CX3CL1-CX3CR1, CD40-CD40LG, and CXCL10-CXCR3 with Bonferroni-corrected p < 0.01
- Heatmap check: significant (red) communication appears at expected sender-receiver positions (e.g., CM4→CM2 and CM4→CM3 in the LN dataset) with no clustering and a blue-white-red ramp
- Benchmark invariants: communication performance evaluated as a dichotomous AUC against literature-supported ground truth; decomposition quality via MSE between reconstructed and original expression matrix

## Limitations

- The accuracy of the GRN directly affects MDIC3's communication results; GRN inference accuracy is an open dependency
- There may be missing information in the right submatrix W when the number of cells exceeds the number of genes, and the original expression matrix may contain noise
- Existing L-R databases (e.g., CellChatDB) are incomplete — orphan receptors like DR6 (TNFRSF21), RELT, TROY, and NGFR, and ligands like IL17D without known receptors, cannot be captured in the L-R extraction step
- No software version numbers are stated for MDIC3, GNIPLR, GENIE3, GRNBoost, CellChat, CellPhoneDB, or iTALK; methods were used with default parameters
- Communication networks among individual cells are difficult to verify empirically, so validation focuses on cell-type-level networks

## Evidence

- [abstract] an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways: "an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways"
- [discussion] MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW: "MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW"
- [discussion] The L-R pairs with corrected p values less than 0.01 were considered significant.: "The L-R pairs with corrected p values less than 0.01 were considered significant."
- [readme] The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell.: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [readme] Note that the first row of the single-cell gene expression data must only contains cell names but not contains a header field.: "Note that the first row of the single-cell gene expression data must only contains cell names but not contains a header field."
- [methods] first calculate the GRN using GNIPLR and then infer the cell-cell communications: "first calculate the GRN using GNIPLR and then infer the cell-cell communications"
