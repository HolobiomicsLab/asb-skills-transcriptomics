---
name: zebrafish-embryo-cell-type-annotation-matching
description: Use when the goal is to infer cell-cell communication from scRNA-seq
  of a species outside human/mouse (e.g., 4 hpf zebrafish embryo) where no prior L-R
  database applies, given a gene-by-cell expression matrix and cell-type labels.
when_to_use_negative:
- The species is human or mouse and a curated L-R database suffices — L-R coexpression
  methods such as CellChat, CellPhoneDB, or iTALK may be more directly interpretable
- The input exp.txt contains a header field in its first row, or the metadata cell
  names do not exactly match the expression file header (README requires exact matching
  and no header field)
- No cell-type labels exist — MDIC3 requires 'existing cell classification labels'
  to go from individual cells to cell types
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MDIC3
  role: Unsupervised Python package that factorizes A=RΣW (R from the GRN, Σ from
    SVD, W=(RΣ)+A) to infer cell-cell communication independent of prior L-R knowledge;
    provides readexp, readlabel, MDIC3_score, MDIC3_scoresave functions
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network (GRN adjacency matrix R) from the expression
    matrix via LASSO projection and lagged regression, with block-calculation step
    and multiprocessing for efficiency
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChat
  role: L-R coexpression-based method whose CellChatDB (human/mouse L-R pairs) is
    contrasted in the paper to show why MDIC3 is needed for non-human/mouse species
    like zebrafish
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:d477115867e2b98a558fb731d01a22d9bad1185306f482a2617ba8c464d301ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# zebrafish-embryo-cell-type-annotation-matching

## Summary

Applies MDIC3, an unsupervised GRN-plus-matrix-decomposition method, to infer cell-cell communication in the 4 hpf zebrafish embryo (GEO: GSM3067189) without prior ligand-receptor knowledge, reproducing the epiblast-centered bidirectional paracrine signaling with mesoderm, germline, and EVL.

## When to use

Use when the goal is to infer cell-cell communication from scRNA-seq of a species outside human/mouse (e.g., 4 hpf zebrafish embryo) where no prior L-R database applies, given a gene-by-cell expression matrix and cell-type labels.

## When NOT to use

- The species is human or mouse and a curated L-R database suffices — L-R coexpression methods such as CellChat, CellPhoneDB, or iTALK may be more directly interpretable
- The input exp.txt contains a header field in its first row, or the metadata cell names do not exactly match the expression file header (README requires exact matching and no header field)
- No cell-type labels exist — MDIC3 requires 'existing cell classification labels' to go from individual cells to cell types

## Inputs

- exp.txt: single-cell gene expression matrix (*.txt; rows = genes, columns = cells; first row contains only cell names, no header field)
- metadata.txt: scRNAseq metadata (*.txt; first column = cells matching exp.txt header exactly, second column = cell-type labels)
- 4 hpf zebrafish embryo scRNA-seq data from GEO: GSM3067189

## Outputs

- celltype_communication.txt: cell-type-level communication matrix (row i, column j = strength of signal sent by cell type i and received by cell type j)
- cellular_communication.txt: single-cell-level communication matrix (row i, column j = strength of signal sent by cell i and received by cell j)
- GRN adjacency matrix from GNIPLR (gene regulatory network)

## How to apply

Download the 4 hpf zebrafish embryo data (GEO: GSM3067189) and format exp.txt as genes (rows) by cells (columns) whose first row contains only cell names, and metadata.txt as cells in column 1 with matching cell-type labels in column 2. Load with lucky.readexp('exp.txt') and lucky.readlabel('metadata.txt'), then infer the GRN via lucky.GRN_GNIPLR(AA, gene_exp, step, process), setting the block-calculation step (e.g., 500; README advises step <= 3000 when genes < 3000) and work processes (e.g., 3) per machine configuration. Run lucky.MDIC3_score(AA, GRN, labels, label_index), which factorizes A=RΣW using the GNIPLR GRN as R, singular value decomposition of A as Σ, and the pseudoinverse solution W=(RΣ)+A, so the sign of Wij gives communication direction and |Wij| gives strength. Save results with lucky.MDIC3_scoresave(ccc_adjacency, type_adjacency, labels) and validate that epiblast shows bidirectional (non-zero strength in both directions) paracrine signaling with mesoderm, germline, and EVL, matching the Figure S7 structure.

## Related tools

- **MDIC3** (Unsupervised Python package that factorizes A=RΣW (R from the GRN, Σ from SVD, W=(RΣ)+A) to infer cell-cell communication independent of prior L-R knowledge; provides readexp, readlabel, MDIC3_score, MDIC3_scoresave functions) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network (GRN adjacency matrix R) from the expression matrix via LASSO projection and lagged regression, with block-calculation step and multiprocessing for efficiency) — https://github.com/zyllluck/GNIPLR
- **CellChat** (L-R coexpression-based method whose CellChatDB (human/mouse L-R pairs) is contrasted in the paper to show why MDIC3 is needed for non-human/mouse species like zebrafish) — https://github.com/sqjin/CellChat

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=results/
```

## Evaluation signals

- celltype_communication.txt shows epiblast with non-zero communication strength in BOTH directions (epiblast→ and →epiblast) for mesoderm, germline, and EVL, reproducing the Figure S7 epiblast-centered structure
- First column of metadata.txt matches exactly with the first row of exp.txt (README requirement), and exp.txt first row contains only cell names with no header field
- Reconstructed matrix from A=RΣW approximates the original expression matrix (low MSE between reconstructed and original single-cell gene expression matrix)
- Signs of Wij in the W matrix yield a consistent direction of communication (directional, not symmetric, adjacency)
- Expression data contains at least 10 cells (README suggestion) and the GRN step parameter was set to <= 3000 for datasets under 3000 genes

## Limitations

- The accuracy of the GRN can affect the results of cell-cell communication for MDIC3 (GRN inference quality is an open dependency)
- There may be some missing information in the right submatrix when the number of cells is greater than the number of genes
- The original single-cell expression matrix may always contain other noise when used as the middle submatrix Σ
- Equation 5 is only applicable when the number of cells and the number of genes are equal and matrix RΣ is invertible (hence the pseudoinverse Equation 6)
- Communication networks of individual cells are difficult to verify; validation is demonstrated at the cell-type level

## Evidence

- [abstract] an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways: "an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways"
- [discussion] The 04hpf zebrafish embryo dataset can be obtained via GEO: GSM3067189.: "The 04hpf zebrafish embryo dataset can be obtained via GEO: GSM3067189."
- [methods] ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index): "ccc_adjacency, type_adjacency = lucky.MDIC3_score(AA, GRN, labels, label_index)"
- [discussion] The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [readme] the first row of the single-cell gene expression data must only contains cell names but not contains a header field: "the first row of the single-cell gene expression data must only contains cell names but not contains a header field"
- [readme] When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less.: "When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less."
