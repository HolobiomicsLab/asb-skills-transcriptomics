---
name: cell-cell-communication-matrix-factorization
description: Use when you have scRNA-seq expression (genes x cells) plus cell labels
  and need species-agnostic cell-cell communication inference, especially when no
  curated L-R database exists for the species or condition-specific shared vs unique
  signals are of interest.
when_to_use_negative:
- The species/condition relies on well-annotated L-R coexpression analysis and curated
  L-R databases suffice — methods like CellChat, CellPhoneDB, or iTALK operate on
  L-R coexpression and may be preferred
- No reliable cell type labels are available and no GRN can be inferred — GRN quality
  directly affects MDIC3 results
- The expression matrix contains fewer than 10 cells, as the README suggests users
  input data containing at least 10 cells
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: MDIC3
  role: Core matrix-decomposition method that factorizes A=RΣW to infer cell-cell
    communication; available as MDIC3.py/MDIC3_LR.py scripts or pip-installable Python
    package
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network (GRN adjacency matrix used as left submatrix
    R) via LASSO projection and lagged regression
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChatDB
  role: Source of human and mouse ligand-receptor gene pairs for the optional L-R
    extraction step (readLRDB/LR_exp)
  repo: https://github.com/sqjin/CellChat
- name: GENIE3 / GRNBoost
  role: Alternative GRN inference tools used to replace the GNIPLR-derived GRN in
    benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:bdebc58daad41d424f1ac6b602c642af01026d83a73ea41be81100cfbc69986e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChatDB
  - GENIE3 / GRNBoost
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# cell-cell-communication-matrix-factorization

## Summary

Infers cell-cell communication from scRNA-seq by factorizing the expression matrix A=RΣW (MDIC3) with a gene regulatory network as the left submatrix R, yielding single-cell and cell-type communication matrices without relying on ligand-receptor databases.

## When to use

Use when you have scRNA-seq expression (genes x cells) plus cell labels and need species-agnostic cell-cell communication inference, especially when no curated L-R database exists for the species or condition-specific shared vs unique signals are of interest.

## When NOT to use

- The species/condition relies on well-annotated L-R coexpression analysis and curated L-R databases suffice — methods like CellChat, CellPhoneDB, or iTALK operate on L-R coexpression and may be preferred
- No reliable cell type labels are available and no GRN can be inferred — GRN quality directly affects MDIC3 results
- The expression matrix contains fewer than 10 cells, as the README suggests users input data containing at least 10 cells

## Inputs

- Single-cell gene expression .txt file with genes as rows and cells as columns, first row containing only cell names (no header field)
- scRNA-seq metadata .txt file with cells in column 1 and cell labels in column 2, cell names matching the expression matrix exactly
- GRN adjacency matrix .txt file (numerical only) when grnchoose='other'
- Optional ligand-receptor database (e.g., CellChatDB Human_LR.txt) for downstream L-R pair extraction

## Outputs

- celltype_communication.txt — cell-type-level communication matrix; value at row i, column j is strength of signal sent from cell type i and received by cell type j
- cellular_communication.txt — single-cell-level communication matrix among individual cells
- ccc_adjacency and type_adjacency matrices from lucky.MDIC3_score
- Optional LRexp table of significant L-R pairs (Bonferroni-corrected p < 0.01) extracted from the communication results

## How to apply

Load the expression matrix (genes as rows, cells as columns, no header field) and metadata with lucky.readexp and lucky.readlabel, ensuring metadata cell names match the expression matrix exactly. Infer the GRN with GNIPLR (LASSO projection plus lagged regression) via lucky.GRN_GNIPLR using a block step size of 500 or less for large gene counts and a multiprocessing pool sized to the machine, or supply an external GRN adjacency matrix with grnchoose='other'. Run lucky.MDIC3_score(AA, GRN, labels, label_index), which factorizes A=RΣW with W=(RΣ)+A solved by pseudoinverse, where the sign of Wij gives communication direction and |Wij| gives strength; output is ccc_adjacency (single-cell) and type_adjacency (cell-type) matrices. Save with lucky.MDIC3_scoresave to produce celltype_communication.txt and cellular_communication.txt, where row i column j is the signal sent by cell type i and received by cell type j. For joint multi-condition analyses (e.g., atopic dermatitis LS vs NL skin from GSE147424), run per condition and compare celltype_communication.txt heatmaps (pheatmap, blue-white-red, no clustering) to identify shared and condition-specific communications.

## Related tools

- **MDIC3** (Core matrix-decomposition method that factorizes A=RΣW to infer cell-cell communication; available as MDIC3.py/MDIC3_LR.py scripts or pip-installable Python package) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network (GRN adjacency matrix used as left submatrix R) via LASSO projection and lagged regression) — https://github.com/zyllluck/GNIPLR
- **CellChatDB** (Source of human and mouse ligand-receptor gene pairs for the optional L-R extraction step (readLRDB/LR_exp)) — https://github.com/sqjin/CellChat
- **GENIE3 / GRNBoost** (Alternative GRN inference tools used to replace the GNIPLR-derived GRN in benchmarking)

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=results/
```

## Evaluation signals

- Matrix decomposition validity: benchmark by mean squared error (MSE) between the reconstructed matrix (RΣW) and the original single-cell gene expression matrix
- Communication results scored as a dichotomous problem against literature-supported ground truth using AUC
- Output schema check: celltype_communication.txt and cellular_communication.txt exist, first row and column are cell labels, and entry (i, j) is the signal sent by i and received by j
- Joint LS/NL validation (GSE147424 atopic dermatitis): heatmap must mark InflameFIB→InflameDC as significant in LS only, with 8 shared immune-cell and 14 shared inflammatory-cell communications matching Figure 4E
- Replaced-GRN robustness: rerun with GRN from GENIE3 or GRNBoost and confirm communication inference still recovers expected interactions

## Limitations

- The accuracy of the GRN can affect the results of cell-cell communication for MDIC3; GRN inference quality is an open dependency
- There may be some missing information in the right submatrix W when the number of cells is greater than the number of genes
- Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible, hence the pseudoinverse is used
- The original single-cell expression matrix may contain noise when used as the middle submatrix Σ
- Existing L-R databases are incomplete — many pairs are undiscovered (orphan receptors DR6, RELT, TROY, NGFR; ligand IL17D without receptor)
- Communication networks among individual cells are difficult to experimentally verify; validation is demonstrated at the cell-type level

## Evidence

- [discussion] MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW: "MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW"
- [discussion] pseudoinverse W: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W:(Equation 6)W=(RΣ)+A"
- [discussion] sign and magnitude interpretation: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [discussion] GNIPLR as GRN source: "Here, we used the GNIPLR111 algorithm (gene network inference on the basis of projection and lagged regression) to investigate GRNs with high accuracy."
- [readme] input format requirement: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [readme] output orientation: "the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type."
- [abstract] unsupervised scope: "an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways"
