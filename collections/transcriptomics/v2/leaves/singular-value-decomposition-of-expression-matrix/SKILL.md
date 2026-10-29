---
name: singular-value-decomposition-of-expression-matrix
description: 'Use when inferring cell-cell communication from a genes×cells scRNA-seq
  expression matrix with MDIC3: SVD of A supplies the singular matrix Σ needed to
  solve the right submatrix W=(RΣ)+A.'
when_to_use_negative:
- Equation 5 (W=A(ΣR)^-1-style direct inversion) — it only applies when the number
  of cells equals the number of genes and RΣ is invertible; use the pseudoinverse
  formulation instead
- When the goal is ligand-receptor-pair-based inference in a species with a curated
  L-R database (e.g. human/mouse via CellChatDB), L-R methods such as CellChat, CellPhoneDB,
  or iTALK may be more appropriate
- When a high-quality GRN cannot be obtained — the accuracy of the GRN directly affects
  MDIC3's cell-cell communication results
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Python package/scripts (MDIC3.py, MDIC3_LR.py) that perform the SVD-based
    matrix decomposition and infer cell-cell communication as W=(RΣ)+A
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the GRN adjacency matrix R (left submatrix) that is combined with the
    SVD-derived singular matrix Σ in the decomposition
  repo: https://github.com/zyllluck/GNIPLR
- name: numpy
  role: Loads an externally computed GRN adjacency matrix from plain-text (numpy.loadtxt)
    for use in the decomposition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:faa463a05273d0bd56c70a536176069c61125fc5a81f7a17c57151efd3422fd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - GNIPLR
  - MDIC3
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# singular-value-decomposition-of-expression-matrix

## Summary

Decompose the m×n single-cell gene expression matrix A by singular value decomposition to obtain the singular (diagonal) matrix Σ that connects genes and cells, serving as the middle submatrix in MDIC3's A=RΣW factorization. This step enables solving the cell-cell communication adjacency matrix W via pseudoinverse when RΣ is not invertible.

## When to use

Use when inferring cell-cell communication from a genes×cells scRNA-seq expression matrix with MDIC3: SVD of A supplies the singular matrix Σ needed to solve the right submatrix W=(RΣ)+A.

## When NOT to use

- Equation 5 (W=A(ΣR)^-1-style direct inversion) — it only applies when the number of cells equals the number of genes and RΣ is invertible; use the pseudoinverse formulation instead
- When the goal is ligand-receptor-pair-based inference in a species with a curated L-R database (e.g. human/mouse via CellChatDB), L-R methods such as CellChat, CellPhoneDB, or iTALK may be more appropriate
- When a high-quality GRN cannot be obtained — the accuracy of the GRN directly affects MDIC3's cell-cell communication results

## Inputs

- Single-cell gene expression matrix A as a genes×cells .txt file (e.g. exp.txt; each row a gene, each column a cell, no header field, first row only cell names)
- m×m GRN adjacency matrix R (from lucky.GRN_GNIPLR or an imported plain-text numeric GRN.txt)
- Cell label metadata (metadata.txt) for downstream cell-type-level communication

## Outputs

- Diagonal singular matrix Σ from SVD of the expression matrix A
- n×n cell-cell communication adjacency matrix W (cellular_communication.txt), where row i / column j encodes signal sent from cell i to cell j
- Cell-type-level communication matrix (celltype_communication.txt) via lucky.MDIC3_score and lucky.MDIC3_scoresave

## How to apply

Load the single-cell expression data (genes as rows, cells as columns, e.g. exp.txt via lucky.readexp) so that A is the m×n matrix of m genes and n cells. Factorize A as A=RΣW where R is the m×m GRN adjacency matrix (computed with lucky.GRN_GNIPLR or imported as a plain-text numeric GRN.txt), and obtain Σ by decomposing A with singular value decomposition — Σ is the diagonal singular matrix connecting genes and cells. Because the number of cells and genes usually differ and RΣ is not invertible, compute the pseudoinverse (RΣ)+ and solve the right submatrix as W=(RΣ)+A (Equation 6), not Equation 5 which requires n=m and invertibility. Interpret W entry-wise: sign of Wij gives communication direction between cells i and j, |Wij| gives strength, yielding an n×n weighted directed adjacency matrix (e.g. 1394×1394 for the test dataset).

## Related tools

- **MDIC3** (Python package/scripts (MDIC3.py, MDIC3_LR.py) that perform the SVD-based matrix decomposition and infer cell-cell communication as W=(RΣ)+A) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the GRN adjacency matrix R (left submatrix) that is combined with the SVD-derived singular matrix Σ in the decomposition) — https://github.com/zyllluck/GNIPLR
- **numpy** (Loads an externally computed GRN adjacency matrix from plain-text (numpy.loadtxt) for use in the decomposition)

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=results_folder
```

## Evaluation signals

- cellular_communication.txt exists and is a square n×n numeric matrix with n equal to the number of cells (e.g. 1394 for the 2000-gene/1394-cell test dataset)
- Row i / column j of the output encodes the communication signal sent from cell i to cell j; spot-check sign (direction) and magnitude (strength) convention of entries
- Matrix decomposition quality can be benchmarked by the mean squared error (MSE) between the reconstructed matrix RΣW and the original expression matrix A
- Performance of the resulting communication inference can be evaluated with AUC against literature-supported ground truth
- Input format check: expression .txt has genes as rows and cells as columns with no header field, and metadata.txt cell names match the expression file's first row exactly

## Limitations

- There may be some missing information in the right submatrix when the number of cells is greater than the number of genes
- The original single-cell expression matrix may contain noise that propagates through the decomposition
- Equation 5 is only applicable when the number of cells equals the number of genes and matrix RΣ is invertible, so the pseudoinverse must be used
- The accuracy of the GRN can affect the results of cell-cell communication for MDIC3
- Communication networks of individual cells are difficult to verify, so validation is typically done at the cell-type level

## Evidence

- [discussion] The singular matrix corresponding to matrix A can be obtained by decomposing matrix A using the singular value decomposition method.: "The singular matrix corresponding to matrix A can be obtained by decomposing matrix A using the singular value decomposition method."
- [discussion] we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W:(Equation 6)W=(RΣ)+A: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W:(Equation 6)W=(RΣ)+A"
- [discussion] Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible.: "Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible."
- [discussion] MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW: "MDIC3 factorizes the matrix A into three matrices R, Σ, and W (Figure 1):(Equation 4)A=RΣW"
- [readme] The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell.: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
