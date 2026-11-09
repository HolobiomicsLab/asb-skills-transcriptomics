---
name: single-cell-communication-inference
description: Use when scRNA-seq expression data (genes x cells) plus cell-type labels
  need unsupervised cell-cell communication inference without relying on predefined
  L-R pairs, e.g., non-human species or benchmarking against CellChat/CellPhoneDB/iTALK.
when_to_use_negative:
- When the analysis goal strictly requires literature-curated ligand-receptor interpretation
  only (e.g., an established human L-R focused workflow with CellChat/CellPhoneDB/iTALK
  defaults) rather than species-agnostic, GRN-based inference.
- When input lacks existing cell classification labels, since MDIC3 infers communication
  networks from individual cells to cell types 'based on existing cell classification
  labels'.
- 'When the GRN cannot be trusted or computed: the article notes ''the accuracy of
  the GRN can affect the results of cell-cell communication for MDIC3'', so a poor-quality
  GRN invalidates the decomposition.'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: MDIC3
  role: 'Core tool: matrix decomposition (W=(R*Sigma)+A) inferring cellular and cell-type
    communication adjacencies from scRNA-seq; run via MDIC3.py or the pip-installable
    package'
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network adjacency matrix used as the left submatrix
    R in the decomposition
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChat
  role: Benchmark comparator (L-R coexpression-based); CellChatDB supplies human and
    mouse L-R genes for MDIC3_LR.py pair extraction
  repo: https://github.com/sqjin/CellChat
- name: GENIE3
  role: Alternative GRN inference tool used to replace the GNIPLR-derived GRN
- name: GRNBoost
  role: Alternative GRN inference tool used to replace the GNIPLR-derived GRN
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:9655560491774e7339623c394da791be69c7fef92e98ea496ab8382eca2a7998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - CellPhoneDB
  - GENIE3
  - GNIPLR
  - GRNBoost
  - MDIC3
  - Python
  - iTALK
  - pheatmap
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# single-cell-communication-inference

## Summary

Infer cell-cell communication from scRNA-seq data using MDIC3, which factorizes the expression matrix A into R*Sigma*W with the GRN adjacency matrix (from GNIPLR) as R and solves W=(R*Sigma)+A as a weighted directed cell-cell communication adjacency matrix, independent of prior ligand-receptor knowledge.

## When to use

Use when scRNA-seq expression data (genes x cells) plus cell-type labels need unsupervised cell-cell communication inference without relying on predefined L-R pairs, e.g., non-human species or benchmarking against CellChat/CellPhoneDB/iTALK.

## When NOT to use

- When the analysis goal strictly requires literature-curated ligand-receptor interpretation only (e.g., an established human L-R focused workflow with CellChat/CellPhoneDB/iTALK defaults) rather than species-agnostic, GRN-based inference.
- When input lacks existing cell classification labels, since MDIC3 infers communication networks from individual cells to cell types 'based on existing cell classification labels'.
- When the GRN cannot be trusted or computed: the article notes 'the accuracy of the GRN can affect the results of cell-cell communication for MDIC3', so a poor-quality GRN invalidates the decomposition.

## Inputs

- exp.txt: single-cell gene expression matrix (*.txt, rows = genes, columns = cells, first row contains only cell names, no header field)
- metadata.txt: scRNA-seq metadata (*.txt, column 1 = cells matching exp.txt first row exactly, column 2 = cell labels); at least 10 cells suggested
- GRN adjacency matrix *.txt file (numeric only, e.g., GRN.txt) when -grnchoose='other'
- Ligand-receptor database file (e.g., Human_LR.txt or mouse CellChatDB L-R file) for MDIC3_LR.py pair extraction

## Outputs

- celltype_communication.txt: cell-type-level communication matrix (row i, column j = strength of signal sent from cell type i and received by cell type j)
- cellular_communication.txt: single-cell n x n communication adjacency matrix (row i, column j = signal sent from cell i to cell j)
- Significant L-R pairs underlying inferred communications (Bonferroni-corrected p < 0.01) from MDIC3_LR.py

## How to apply

Format the expression data as exp.txt (genes as rows, cells as columns, cell names only in the first row, no header field) and metadata.txt (cell in column 1, cell label in column 2, matching the first row of exp.txt exactly). Infer the GRN with lucky.GRN_GNIPLR(AA, gene_exp, step, process), setting the block-calculation step to the number of genes when genes < 3000, or a value of 3000 or less otherwise, and choosing the number of multiprocessing work processes per machine configuration; alternatively import an external GRN adjacency matrix via -grnchoose='other' (MDIC3 is not limited to GNIPLR and accepts GENIE3 or GRNBoost results). Run lucky.MDIC3_score, which performs singular value decomposition of A to obtain Sigma, computes the pseudoinverse (R*Sigma)+ and solves W=(R*Sigma)+A (Equation 6, valid even when cells and genes differ and R*Sigma is not invertible), interpreting sign(Wij) as communication direction and |Wij| as strength. Persist results with lucky.MDIC3_scoresave to produce celltype_communication.txt and cellular_communication.txt. Optionally extract L-R pairs with MDIC3_LR.py using the CellChatDB-derived human or mouse L-R file, filtering with Bonferroni-corrected p < 0.01.

## Related tools

- **MDIC3** (Core tool: matrix decomposition (W=(R*Sigma)+A) inferring cellular and cell-type communication adjacencies from scRNA-seq; run via MDIC3.py or the pip-installable package) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network adjacency matrix used as the left submatrix R in the decomposition) — https://github.com/zyllluck/GNIPLR
- **CellChat** (Benchmark comparator (L-R coexpression-based); CellChatDB supplies human and mouse L-R genes for MDIC3_LR.py pair extraction) — https://github.com/sqjin/CellChat
- **GENIE3** (Alternative GRN inference tool used to replace the GNIPLR-derived GRN)
- **GRNBoost** (Alternative GRN inference tool used to replace the GNIPLR-derived GRN)

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=results_output_fold
```

## Evaluation signals

- cellular_communication.txt exists, is a square n x n numeric matrix (n = number of cells, e.g., 1394 for the 2000-gene/1394-cell test data), with row i / column j encoding signal sent from cell i to cell j
- Spot-check that the sign of Wij encodes communication direction and |Wij| encodes strength per the MDIC3 convention
- Benchmark AUC against literature-supported ground truth reproduces published values (e.g., MDIC3 0.801 vs CellChat 0.247 on human islet GSE116753; 0.711 vs 0.38 on mouse brain GSE60361)
- Extracted L-R pairs (Bonferroni-corrected p < 0.01) recover known signaling such as SST-SSTR2 (delta-to-alpha, islets) and Cx3cl1-Cx3cr1 / Apoe-Lrp1 (interneuron-microglia, brain)
- MSE between the reconstructed matrix and the original single-cell gene expression matrix confirms the matrix decomposition is faithful

## Limitations

- MDIC3 results depend on GRN accuracy: 'the accuracy of the GRN can affect the results of cell-cell communication for MDIC3.'
- Equation 5 is only applicable when the number of cells equals the number of genes and R*Sigma is invertible; the pseudoinverse (Equation 6) is required otherwise, and some information may be missing in the right submatrix when cells exceed genes.
- The original single-cell expression matrix may contain noise when used as the middle submatrix.
- Communication networks of individual cells are difficult to verify, so validation is demonstrated mainly at the cell-type level.
- Existing L-R databases are incomplete (e.g., orphan receptors DR6, RELT, TROY, NGFR), which constrains L-R-based pair extraction even though MDIC3 inference itself is L-R independent.

## Evidence

- [discussion] Matrix decomposition core: "we adopt the pseudoinverse matrix (RΣ)+ to solve the matrix W:(Equation 6)W=(RΣ)+A"
- [discussion] Interpretation of W: "The sign of Wij represents the communication direction between the cell i and j, and |Wij| denotes the communication strength"
- [discussion] GNIPLR as GRN source: "Here, we used the GNIPLR111 algorithm (gene network inference on the basis of projection and lagged regression) to investigate GRNs with high accuracy."
- [readme] Input format: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [readme] Step parameter guidance: "When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less."
- [discussion] L-R pair significance filter: [evidence span withheld — anchor status: not_found]
- [abstract] Species independence rationale: "an unsupervised tool to investigate cell-cell communication in any species, and the results are not limited by specific L-R pairs or signaling pathways"
