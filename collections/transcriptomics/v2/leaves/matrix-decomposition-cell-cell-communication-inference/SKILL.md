---
name: matrix-decomposition-cell-cell-communication-inference
description: Use when analyzing scRNA-seq data (exp.txt gene-by-cell matrix plus metadata.txt
  cell labels) for cell-cell communication in species beyond human/mouse, or when
  no prior L-R database should constrain the inference.
when_to_use_negative:
- The input lacks existing cell classification labels (MDIC3 infers communication
  from individual cells to cell types based on existing labels; users are also advised
  to include at least 10 cells).
- The analysis must be grounded in a curated ligand-receptor database as the primary
  evidence; in that case L-R-coexpression methods such as CellChat, CellPhoneDB, or
  iTALK are the appropriate choice.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MDIC3
  role: Python package/implementation of the matrix decomposition (A=RΣW) that infers
    cell-cell communication from the GRN and expression matrix
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network whose adjacency matrix becomes the left
    submatrix R; run via lucky.GRN_GNIPLR with step and process parameters
  repo: https://github.com/zyllluck/GNIPLR
- name: GENIE3
  role: Alternative GRN inference tool; its numerical adjacency matrix can be substituted
    for the GNIPLR GRN via -grnchoose='other' -grn=GRN.txt to test robustness
- name: GRNBoost
  role: Second alternative GRN inference tool used with default parameters to replace
    the GNIPLR GRN and benchmark communication-matrix stability
- name: CellChat
  role: L-R coexpression-based comparator method; its CellChatDB supplies human and
    mouse ligand-receptor genes for benchmarking against MDIC3
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:fdc6088a2cf714f2403aebc959df709cb1c6dc32fd3c2d62eb90f2efc3793f51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - GENIE3
  - GNIPLR
  - GRNBoost
  - MDIC3
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# matrix-decomposition-cell-cell-communication-inference

## Summary

MDIC3 infers cell-cell communication from scRNA-seq by factorizing the gene expression matrix A into R*Sigma*W, where R is a gene regulatory network (GRN) adjacency matrix computed with GNIPLR, and interpreting W's elements as directed communication strengths — independent of ligand-receptor databases and applicable to any species.

## When to use

Use when analyzing scRNA-seq data (exp.txt gene-by-cell matrix plus metadata.txt cell labels) for cell-cell communication in species beyond human/mouse, or when no prior L-R database should constrain the inference.

## When NOT to use

- The input lacks existing cell classification labels (MDIC3 infers communication from individual cells to cell types based on existing labels; users are also advised to include at least 10 cells).
- The analysis must be grounded in a curated ligand-receptor database as the primary evidence; in that case L-R-coexpression methods such as CellChat, CellPhoneDB, or iTALK are the appropriate choice.

## Inputs

- exp.txt: single-cell gene expression matrix as a .txt file, rows = genes, columns = cells, first row containing cell names only (no header field)
- metadata.txt: cell labels as a .txt file, first column = cells (matching the expression file header exactly), second column = cell-type labels
- Optional external GRN adjacency matrix *.txt (numerical values only) when -grnchoose='other'

## Outputs

- celltype_communication.txt: cell-type-level communication matrix; value in row i, column j = strength of signal sent from cell type i and received by cell type j
- cellular_communication.txt: single-cell-level communication matrix with the same sender/receiver row-column convention

## How to apply

Load the single-cell expression file with lucky.readexp('exp.txt') and labels with lucky.readlabel('metadata.txt'), where the metadata first column must exactly match the expression file's first row. Infer the GRN with lucky.GRN_GNIPLR(AA, gene_exp, step, process), setting the block-calculation step to the gene count (or ≤3000 when genes are under 3000) and the process count per machine configuration; alternatively import an external GRN as a numerical adjacency matrix *.txt via -grnchoose='other' -grn=GRN.txt. Run lucky.MDIC3_score, which factorizes A=RΣW with R as the GRN adjacency matrix (left submatrix), Σ from singular value decomposition, and W=(RΣ)+A via the pseudoinverse; the sign of Wij gives communication direction and |Wij| the strength. Save results with lucky.MDIC3_scoresave to produce celltype_communication.txt and cellular_communication.txt, where row i / column j indicates signal sent by cell type i and received by cell type j.

## Related tools

- **MDIC3** (Python package/implementation of the matrix decomposition (A=RΣW) that infers cell-cell communication from the GRN and expression matrix) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network whose adjacency matrix becomes the left submatrix R; run via lucky.GRN_GNIPLR with step and process parameters) — https://github.com/zyllluck/GNIPLR
- **GENIE3** (Alternative GRN inference tool; its numerical adjacency matrix can be substituted for the GNIPLR GRN via -grnchoose='other' -grn=GRN.txt to test robustness)
- **GRNBoost** (Second alternative GRN inference tool used with default parameters to replace the GNIPLR GRN and benchmark communication-matrix stability)
- **CellChat** (L-R coexpression-based comparator method; its CellChatDB supplies human and mouse ligand-receptor genes for benchmarking against MDIC3) — https://github.com/sqjin/CellChat

## Examples

```
python MDIC3.py -exp=scRNA_expression_file -label=cell_label_file -grnchoose='GNIPLR' -process=3 -step=500 -out=results_output_folder
```

## Evaluation signals

- celltype_communication.txt has dimensions equal to the number of cell types (e.g., 7x7 for the 7-type test dataset) with identical sender/receiver orientation across all runs (row i = sender, column j = receiver).
- The communication table covers all directed cell-type pairs (e.g., 49 pairs for 7 cell types) under each GRN tool used.
- Substituting GENIE3 or GRNBoost for the GNIPLR GRN yields communication matrices comparable element-wise to the GNIPLR result (pairwise correlations and per-pair strength differences compiled, e.g., into grn_tool_comparison_table.csv).
- Reproducing published patterns: 41 shared communications between E13.5 and E14.5 mouse skin with an E14.5-specific bidirectional spinous–basal-P signal, dermal-to-epidermal community-center shift (Figures 4B–4D), and epiblast-centered bidirectional paracrine signaling at 4 hpf zebrafish (Figure S7).
- Matrix decomposition quality can be checked via the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix.

## Limitations

- The accuracy of the GRN can affect the results of cell-cell communication for MDIC3, so GRN inference quality is an open dependency.
- There may be some missing information in the right submatrix when the number of cells is greater than the number of genes.
- Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible (hence the pseudoinverse form W=(RΣ)+A is used).
- The original single-cell gene expression matrix may contain other noise, and communication networks of individual cells are difficult to verify.

## Evidence

- [discussion] Factorization definition: "MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW"
- [discussion] Left submatrix is the GRN: "MDIC3 uses the GRN adjacency matrix of matrix A as the left submatrix R in Equation 4."
- [discussion] W solved by pseudoinverse: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W:(Equation 6)W=(RΣ)+A"
- [discussion] Interpretation of W elements: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [readme] Output file convention: "'celltype_communication.txt' is the result of communication among different cell types, while the first column and the first row represent different cell labels."
- [readme] CLI usage for GNIPLR path: "python MDIC3.py -exp=scRNA_expression_file -label=cell_label_file -grnchoose='GNIPLR' -process=process_value -step=GRN_calculation_step -out=results_output_fold"
