---
name: directed-communication-matrix-interpretation
description: Use when you have run MDIC3 (or an equivalent A=RΣW decomposition with
  a GRN-derived left submatrix R) on scRNA-seq data and must convert the resulting
  W matrix entries into directed, sender-to-receiver communication strengths between
  cells or cell types.
when_to_use_negative:
- 'You need explicit ligand-receptor identities: MDIC3 output is not limited to specific
  L-R pairs or signaling pathways, so use CellChat/CellPhoneDB/iTALK when named L-R
  pairs are required'
- The species lacks curated ligand-receptor databases for L-R-based tools — that is
  precisely the case where this L-R-independent interpretation should be used instead
- The expression matrix has fewer than 10 cells per the README suggestion, making
  cell/cell-type communication aggregation unreliable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MDIC3
  role: Performs the A=RΣW matrix decomposition whose right submatrix W is interpreted
    as the directed communication matrix; outputs cell- and cell-type-level communication
    matrices via lucky.MDIC3_score and lucky.MDIC3_scoresave
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network adjacency matrix used as the left submatrix
    R of the decomposition, anchoring directionality of communication in gene regulatory
    relationships
  repo: https://github.com/zyllluck/GNIPLR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:4a5ea4034f51c0aeec6fbb0472ca97e76e8b497027bfcfa24793d5e800eea938
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# directed-communication-matrix-interpretation

## Summary

Interpret MDIC3's matrix factorization output (A=RΣW) as a directed cell-cell communication matrix, reading the sign of Wij as communication direction and |Wij| as communication strength from sender to receiver cell types.

## When to use

Use when you have run MDIC3 (or an equivalent A=RΣW decomposition with a GRN-derived left submatrix R) on scRNA-seq data and must convert the resulting W matrix entries into directed, sender-to-receiver communication strengths between cells or cell types.

## When NOT to use

- You need explicit ligand-receptor identities: MDIC3 output is not limited to specific L-R pairs or signaling pathways, so use CellChat/CellPhoneDB/iTALK when named L-R pairs are required
- The species lacks curated ligand-receptor databases for L-R-based tools — that is precisely the case where this L-R-independent interpretation should be used instead
- The expression matrix has fewer than 10 cells per the README suggestion, making cell/cell-type communication aggregation unreliable

## Inputs

- Single-cell gene expression matrix as .txt (rows = genes, columns = cells, first row cell names only, e.g. exp.txt)
- Cell-type labels metadata file (.txt, first column = cells, second column = labels, e.g. metadata.txt)
- GRN adjacency matrix R from GNIPLR (GRN_GNIPLR output) as .txt of numeric values
- Cell labels and label index from lucky.readlabel('metadata.txt')

## Outputs

- cellular_communication.txt: single-cell directed communication matrix (row i, column j = strength of signal sent by cell i, received by cell j)
- celltype_communication.txt: cell-type directed communication matrix (row i, column j = strength of signal sent by cell type i, received by cell type j)

## How to apply

Factorize the single-cell expression matrix A as A=RΣW, using the GRN adjacency matrix computed by GNIPLR as the left submatrix R, the singular value decomposition of A as Σ, and solving the right submatrix via the pseudoinverse W=(RΣ)+A. For each entry Wij, take the sign to represent the communication direction between cell i and cell j and the absolute value to represent communication strength. Aggregate the single-cell communication matrix (cellular_communication.txt) to a cell-type matrix (celltype_communication.txt) with lucky.MDIC3_score and lucky.MDIC3_scoresave, where row i, column j of the cell-type matrix gives the strength of the signal sent from the i-th cell type and received by the j-th cell type. Bidirectional paracrine signaling is declared when strength is non-zero in both directions (e.g., epiblast↔mesoderm, epiblast↔germline, epiblast↔EVL in the 4 hpf zebrafish embryo case).

## Related tools

- **MDIC3** (Performs the A=RΣW matrix decomposition whose right submatrix W is interpreted as the directed communication matrix; outputs cell- and cell-type-level communication matrices via lucky.MDIC3_score and lucky.MDIC3_scoresave) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network adjacency matrix used as the left submatrix R of the decomposition, anchoring directionality of communication in gene regulatory relationships) — https://github.com/zyllluck/GNIPLR

## Examples

```
ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index); lucky.MDIC3_scoresave(ccc_adjacency, type_adjacency, labels)
```

## Evaluation signals

- Verify the celltype_communication.txt matrix is square with first row and column equal to the cell-type labels, and values in row i, column j representing sender i to receiver j
- Directionality check: signal strength should be non-zero in both directions for known bidirectional pairs (e.g., epiblast and mesoderm, germline, EVL in the 4 hpf zebrafish embryo dataset, GSM3067189), matching the epiblast-centered structure of Figure S7
- Confirm the decomposition reconstructs A by benchmarking the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix
- Check W was computed via the pseudoinverse (RΣ)+A, appropriate when cell and gene counts are unequal (Equation 5 only applies when numbers are equal and RΣ is invertible)
- Cross-validate inferred dominant communication directions against literature-supported ground truth using AUC

## Limitations

- Possible missing information in the right submatrix when the number of cells is greater than the number of genes
- The accuracy of the GRN from GNIPLR can affect the cell-cell communication results of MDIC3
- The original single-cell expression matrix may contain noise that propagates through the decomposition
- Communication networks of individual cells are difficult to verify, so interpretation is demonstrated mainly at the cell-type level

## Evidence

- [discussion] The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [discussion] MDIC3 uses the GRN adjacency matrix of matrix A as the left submatrix R in Equation 4.: "MDIC3 uses the GRN adjacency matrix of matrix A as the left submatrix R in Equation 4."
- [discussion] we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W"
- [readme] the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type.: "the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type."
- [methods] ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index): "ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index)"
