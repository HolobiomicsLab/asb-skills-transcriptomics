---
name: pseudoinverse-matrix-computation
description: Use when solving a matrix factorization W=(RΣ)+A where the GRN adjacency
  matrix R (m×m) times the singular matrix Σ of the expression matrix A (m×n) is not
  invertible, so W must be recovered via the pseudoinverse.
when_to_use_negative:
- Equation 5 is applicable only when the number of cells equals the number of genes
  and RΣ is invertible — but in that case the pseudoinverse shortcut is unnecessary;
  the pseudoinverse route (Equation 6) is what handles the general case
- GRN quality is poor or unknown, since the accuracy of the GRN can affect the results
  of cell-cell communication for MDIC3
- The analysis requires interpreting specific ligand-receptor pairs directly from
  the decomposition alone; MDIC3 needs a separate L-R extraction step (readLRDB/LR_exp
  against e.g. CellChatDB Human_LR.txt)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Python package/script (MDIC3.py) implementing the W=(RΣ)+A pseudoinverse solve
    to infer cell-cell communication
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the m×m GRN adjacency matrix R used as the left submatrix prior to
    the pseudoinverse computation
  repo: https://github.com/zyllluck/GNIPLR
- name: GRNBoost
  role: Alternative GRN inference tool that can replace GNIPLR to produce R for the
    decomposition
- name: GENIE3
  role: Alternative GRN inference tool that can replace GNIPLR to produce R for the
    decomposition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:c874f2d7fe83e1342d1a3821d30ed6805c51c8c50b1737dffc44ece05716fad6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - GENIE3
  - GNIPLR
  - GRNBoost
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pseudoinverse-matrix-computation

## Summary

Computes the Moore–Penrose pseudoinverse of the product RΣ to solve the right submatrix W=(RΣ)+A in MDIC3, yielding an n×n cell-cell communication adjacency matrix from an m×n single-cell expression matrix. This step is required because RΣ is generally non-invertible when the number of cells and genes differ.

## When to use

Use when solving a matrix factorization W=(RΣ)+A where the GRN adjacency matrix R (m×m) times the singular matrix Σ of the expression matrix A (m×n) is not invertible, so W must be recovered via the pseudoinverse.

## When NOT to use

- Equation 5 is applicable only when the number of cells equals the number of genes and RΣ is invertible — but in that case the pseudoinverse shortcut is unnecessary; the pseudoinverse route (Equation 6) is what handles the general case
- GRN quality is poor or unknown, since the accuracy of the GRN can affect the results of cell-cell communication for MDIC3
- The analysis requires interpreting specific ligand-receptor pairs directly from the decomposition alone; MDIC3 needs a separate L-R extraction step (readLRDB/LR_exp against e.g. CellChatDB Human_LR.txt)

## Inputs

- m×n single-cell gene expression matrix A (genes×cells, from exp.txt via lucky.readexp)
- m×m GRN adjacency matrix R (from GNIPLR or an external numeric .txt file, e.g. GRN.txt)
- Diagonal singular matrix Σ from SVD of A
- Cell labels and label_index from metadata.txt via lucky.readlabel

## Outputs

- n×n weighted directed cell-cell communication adjacency matrix W (cellular_communication.txt)
- Cell-type-level communication matrix (celltype_communication.txt)

## How to apply

First decompose the m×n single-cell expression matrix A via singular value decomposition to obtain the diagonal singular matrix Σ connecting genes and cells; the left submatrix R is the m×m GRN adjacency matrix (computed with GNIPLR via lucky.GRN_GNIPLR, e.g. step=500, process=3 for the 2000-gene/1394-cell test data, or imported as a plain-text numeric matrix with numpy.loadtxt when grnchoose='other'). Because Equation 5 is only applicable when the number of cells equals the number of genes and RΣ is invertible, compute the pseudoinverse of the product RΣ and solve W=(RΣ)+A (Equation 6). Interpret W entry-wise: the sign of Wij encodes the communication direction between cells i and j and |Wij| encodes communication strength, producing an n×n weighted directed adjacency matrix. Persist the result with lucky.MDIC3_scoresave after computing cell- and cell-type-level communication via lucky.MDIC3_score.

## Related tools

- **MDIC3** (Python package/script (MDIC3.py) implementing the W=(RΣ)+A pseudoinverse solve to infer cell-cell communication) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the m×m GRN adjacency matrix R used as the left submatrix prior to the pseudoinverse computation) — https://github.com/zyllluck/GNIPLR
- **GRNBoost** (Alternative GRN inference tool that can replace GNIPLR to produce R for the decomposition)
- **GENIE3** (Alternative GRN inference tool that can replace GNIPLR to produce R for the decomposition)

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='other' -grn=GRN.txt -out=results
```

## Evaluation signals

- Verify cellular_communication.txt exists and is a square n×n numeric matrix with n equal to the number of cells (e.g., 1394 for the test dataset)
- Check row i / column j convention: entries encode the communication signal sent from cell i and received by cell j
- Spot-check that the sign of Wij reflects communication direction and |Wij| reflects communication strength as documented
- Benchmark the decomposition by computing the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix
- Confirm the metadata first column matches exactly with the first row of the expression data before running, and that at least 10 cells are supplied

## Limitations

- Possible missing information in the right submatrix when the number of cells exceeds the number of genes
- The accuracy of the GRN can affect the results of cell-cell communication for MDIC3
- The original single-cell expression matrix may always contain other noise when used as the middle submatrix
- Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible, motivating the pseudoinverse (Equation 6)

## Evidence

- [discussion] we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W"
- [discussion] Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible.: "Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible."
- [discussion] The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [readme] users must provide the calculated GRN results, which must be a *.txt file containing only the numerical results of the GRN adjacency matrix: "users must provide the calculated GRN results, which must be a *.txt file containing only the numerical results of the GRN adjacency matrix"
- [discussion] calculating the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix: "calculating the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix"
