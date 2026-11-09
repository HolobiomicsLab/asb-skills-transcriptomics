---
name: grn-inference-gniplr
description: Use when a scRNA-seq expression matrix (genes x cells, .txt, rows = genes,
  columns = cells) must be converted into a GRN adjacency matrix to serve as the left
  submatrix R in MDIC3, or when benchmarking GNIPLR against GENIE3/GRNBoost as GRN
  sources.
when_to_use_negative:
- The downstream method requires predefined ligand-receptor interactions or a species-specific
  L-R database (e.g., CellChatDB) — GRN-based inference like GNIPLR/MDIC3 is chosen
  precisely to avoid L-R dependence, and vice versa L-R tools are the wrong choice
  for species lacking L-R annotations
- The expression data is a time-series where Equation 2-style direct inference is
  applicable and time ordering is available — GNIPLR constructs pseudo-time via projection
  rather than using real time-series data
- The input is not a raw gene x cell expression matrix (e.g., it is already a GRN,
  a communication matrix, or a feature table) — the GRN inference step would be redundant
  or invalid
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: GNIPLR
  role: Primary GRN inference algorithm; computes the gene regulatory network adjacency
    matrix from the expression matrix via LASSO projection, linear projection approximation,
    and lagged regression, invoked as lucky.GRN_GNIPLR or python MDIC3.py -grnchoose='GNIPLR'
  repo: https://github.com/zyllluck/GNIPLR
- name: MDIC3
  role: 'Consumer of the GRN: uses the GRN adjacency matrix as the left submatrix
    R in A = RΣW to infer cell-cell communication; wraps GNIPLR via -grnchoose=''GNIPLR''
    and accepts external GRNs via -grnchoose=''other'''
  repo: https://github.com/LYxiaotai/MDIC3
- name: GENIE3
  role: Alternative GRN inference tool used with default parameters to replace the
    GNIPLR-derived GRN and test robustness of the downstream communication matrices
- name: GRNBoost
  role: Second alternative GRN inference tool used with default parameters for the
    same substitution robustness check
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:2cc6edc86fb77e1b337593336e6fe2c37dc28e469bbe0706a29d14741a4c5be4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zyllluck/GNIPLR
  tools:
  - GENIE3
  - GNIPLR
  - GRNBoost
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# grn-inference-gniplr

## Summary

Infer a gene regulatory network (GRN) adjacency matrix from single-cell gene expression data using GNIPLR (gene networks inference based on projection and lagged regression), as used by MDIC3 to supply the left submatrix R of its matrix decomposition. This skill also covers substituting alternative GRN tools (GENIE3, GRNBoost) at the same interface to test robustness of downstream communication inference.

## When to use

Use when a scRNA-seq expression matrix (genes x cells, .txt, rows = genes, columns = cells) must be converted into a GRN adjacency matrix to serve as the left submatrix R in MDIC3, or when benchmarking GNIPLR against GENIE3/GRNBoost as GRN sources.

## When NOT to use

- The downstream method requires predefined ligand-receptor interactions or a species-specific L-R database (e.g., CellChatDB) — GRN-based inference like GNIPLR/MDIC3 is chosen precisely to avoid L-R dependence, and vice versa L-R tools are the wrong choice for species lacking L-R annotations
- The expression data is a time-series where Equation 2-style direct inference is applicable and time ordering is available — GNIPLR constructs pseudo-time via projection rather than using real time-series data
- The input is not a raw gene x cell expression matrix (e.g., it is already a GRN, a communication matrix, or a feature table) — the GRN inference step would be redundant or invalid

## Inputs

- Single-cell gene expression matrix as *.txt (each row a gene, each column a cell; first row contains cell names without a header field), e.g., exp.txt with 2000 genes x 1394 cells
- scRNA-seq metadata *.txt (first column cells, second column cell labels; cell order matching the expression matrix first row), e.g., metadata.txt with 7 cell types
- Optional: externally computed GRN adjacency matrix as *.txt containing only numerical values (e.g., GRN.txt from GENIE3 or GRNBoost)

## Outputs

- GRN adjacency matrix (genes x genes numerical *.txt) usable as the left submatrix R in MDIC3's decomposition A = RΣW
- Downstream cell-type communication matrix celltype_communication.txt (row i = sender cell type, column j = receiver) and single-cell communication matrix cellular_communication.txt from lucky.MDIC3_score
- Robustness comparison table (e.g., grn_tool_comparison_table.csv) of per-pair communication strengths and pairwise differences across GNIPLR/GENIE3/GRNBoost-derived matrices

## How to apply

Load the single-cell expression file (e.g., exp.txt, 2000 genes x 1394 cells) and metadata with lucky.readexp and lucky.readlabel, then compute the GRN via lucky.GRN_GNIPLR(AA, gene_exp, step, process); in the MDIC3 example, step=500 and process=3. GNIPLR works by projecting gene data twice using the LASSO projection algorithm and linear projection approximation to produce a linear, monotonous pseudo-time series, then determining regulation direction with lagged regression analyses, so no time-series input is required. Because large gene counts make GRN computation slow, MDIC3 accelerates it by blocking the target GRN matrix (set '-step' to the gene count or a value <= 3000 when genes number fewer than 3000) and by parallelizing with the Python multiprocessing package (choose 'process' per machine configuration). To substitute another tool, run GENIE3 or GRNBoost with default parameters, write only the numerical adjacency matrix to a *.txt file, and import it through -grnchoose='other' -grn=GRN.txt. The resulting GRN adjacency matrix is used by MDIC3 as the left submatrix R in the decomposition A = RΣW, after which lucky.MDIC3_score produces cell-type and single-cell communication matrices. Validate by confirming the GRN file contains only numerical adjacency values with dimensions equal to the gene count, and by comparing downstream 7x7 cell-type communication matrices across tools for identical dimensions and orientation (row = sender, column = receiver).

## Related tools

- **GNIPLR** (Primary GRN inference algorithm; computes the gene regulatory network adjacency matrix from the expression matrix via LASSO projection, linear projection approximation, and lagged regression, invoked as lucky.GRN_GNIPLR or python MDIC3.py -grnchoose='GNIPLR') — https://github.com/zyllluck/GNIPLR
- **MDIC3** (Consumer of the GRN: uses the GRN adjacency matrix as the left submatrix R in A = RΣW to infer cell-cell communication; wraps GNIPLR via -grnchoose='GNIPLR' and accepts external GRNs via -grnchoose='other') — https://github.com/LYxiaotai/MDIC3
- **GENIE3** (Alternative GRN inference tool used with default parameters to replace the GNIPLR-derived GRN and test robustness of the downstream communication matrices)
- **GRNBoost** (Second alternative GRN inference tool used with default parameters for the same substitution robustness check)

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=3 -step=500 -out=results_output_fold
```

## Evaluation signals

- The GRN output *.txt contains only numerical adjacency values with dimensions equal to the number of genes (2000 x 2000 for the example exp.txt)
- All downstream cell-type communication matrices share identical 7x7 dimensions and orientation (row i = sender, column j = receiver per the celltype_communication.txt convention)
- The robustness comparison table contains communication strengths for all 49 directed cell-type pairs under each of the three GRN tools (GNIPLR, GENIE3, GRNBoost)
- Pairwise correlations and difference magnitudes of communication strengths between the same sender/receiver cell-type pairs are computed element-wise across the three tools' matrices
- For GNIPLR via MDIC3.py, the '-step' parameter is set to the gene count or ≤3000 when the gene number is below 3000, and '-process' matches the machine's multiprocessing capacity

## Limitations

- GRN quality affects MDIC3 results: the accuracy of the GRN can affect the results of cell-cell communication, so GRN inference accuracy is an open dependency
- No software version numbers are stated for GNIPLR, GENIE3, GRNBoost, or MDIC3; the paper states the methods were used with default parameters
- Different data may need different GNIPLR parameters — the public gniplr.py only provides one data parameter and other parameters must be requested from the author
- Equation 5 of the underlying decomposition is only applicable when the number of cells equals the number of genes and RΣ is invertible; the pseudoinverse formulation W=(RΣ)+A is used instead, though some information may be missing in the right submatrix when cells exceed genes

## Evidence

- [readme] GNIPLR projected gene data twice using the LASSO projection algorithm and the linear projection approximation to produce a linear and monotonous pseudo-time series, and then determined the direction of regulation in combination with lagged regression analyses: "GNIPLR projected gene data twice using the LASSO projection algorithm and the linear projection approximation to produce a linear and monotonous pseudo-time series, and then determined the direction"
- [discussion] MDIC3 uses the GRN adjacency matrix of matrix A as the left submatrix R in Equation 4.: "MDIC3 uses the GRN adjacency matrix of matrix A as the left submatrix R in Equation 4."
- [discussion] We used two regulatory network inference tools (GENIE3118 and GRNBoost119) to replace the GRN from GNIPLR: "We used two regulatory network inference tools (GENIE3118 and GRNBoost119) to replace the GRN from GNIPLR"
- [readme] the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type.: "the value in row i and column j indicates the strength of the communication signal sent from the i-th cell type and received by the j-th cell type"
- [readme] When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less.: "When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less"
