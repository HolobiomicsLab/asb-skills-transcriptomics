---
name: grn-inference-benchmarking-with-genie3-and-grnboost
description: Use when a workflow depends on a GRN inferred from a single-cell gene
  expression matrix and you need to test whether the downstream results (e.g., cell-type
  communication matrices) are robust to swapping the GRN inference algorithm.
when_to_use_negative:
- The downstream method requires predefined ligand-receptor pairs or signaling pathway
  knowledge (e.g., CellChat, CellPhoneDB, iTALK workflows) rather than an inferred
  GRN adjacency matrix
- You have no cell classification labels or fewer than 10 cells in the input expression
  data, so MDIC3's cell-to-cell-type communication inference cannot be run
- The goal is to evaluate GRN inference accuracy against a known gold-standard network
  directly, rather than assessing robustness of a downstream communication result
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GNIPLR
  role: Reference GRN inference algorithm producing the baseline adjacency matrix
    used by MDIC3
  repo: https://github.com/zyllluck/GNIPLR
- name: GENIE3
  role: Alternative GRN inference tool (default parameters) whose adjacency matrix
    replaces the GNIPLR GRN in MDIC3
- name: GRNBoost
  role: Alternative GRN inference tool (default parameters) whose adjacency matrix
    replaces the GNIPLR GRN in MDIC3
- name: MDIC3
  role: Downstream method that consumes the GRN adjacency matrix as left submatrix
    R and outputs cell-type communication matrices
  repo: https://github.com/LYxiaotai/MDIC3
- name: Python
  role: Language used to run MDIC3.py/GNIPLR scripts and perform element-wise matrix
    comparisons
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:0d2249b68ca1d3d48ca1d7bbfdb44c2bb2487a469709a473b900ab689dd56fdd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# grn-inference-benchmarking-with-genie3-and-grnboost

## Summary

Benchmark a gene regulatory network (GRN) inference step by substituting alternative tools (GENIE3, GRNBoost, in place of GNIPLR) into a downstream pipeline (MDIC3 cell-cell communication inference) and comparing resulting outputs element-wise to test robustness.

## When to use

Use when a workflow depends on a GRN inferred from a single-cell gene expression matrix and you need to test whether the downstream results (e.g., cell-type communication matrices) are robust to swapping the GRN inference algorithm.

## When NOT to use

- The downstream method requires predefined ligand-receptor pairs or signaling pathway knowledge (e.g., CellChat, CellPhoneDB, iTALK workflows) rather than an inferred GRN adjacency matrix
- You have no cell classification labels or fewer than 10 cells in the input expression data, so MDIC3's cell-to-cell-type communication inference cannot be run
- The goal is to evaluate GRN inference accuracy against a known gold-standard network directly, rather than assessing robustness of a downstream communication result

## Inputs

- Single-cell gene expression matrix as *.txt (rows = genes, columns = cells; first row contains cell names only, no header field), e.g., 2000 genes x 1394 cells
- scRNAseq metadata as *.txt (first column = cell names matching the expression file's first row exactly, second column = cell labels), e.g., 7 cell types
- GRN adjacency matrix computed by GENIE3 (default parameters) as a *.txt file containing only numerical values
- GRN adjacency matrix computed by GRNBoost (default parameters) as a *.txt file containing only numerical values

## Outputs

- Reference cell-type communication matrix (celltype_communication.txt, 7x7, row = sender, column = receiver) from GNIPLR-based MDIC3
- Cell-type communication matrix from GENIE3-substituted MDIC3 run
- Cell-type communication matrix from GRNBoost-substituted MDIC3 run
- Comparison table (e.g., grn_tool_comparison_table.csv) of per-pair communication strengths and pairwise differences for all 49 directed cell-type pairs

## How to apply

First run the reference pipeline: load exp.txt (genes x cells, first row = cell names, no header field) and metadata.txt (cells + cell labels) via lucky.readexp/lucky.readlabel, infer the GRN with lucky.GRN_GNIPLR (e.g., step=500, process=3), and run lucky.MDIC3_score to obtain the cell-type communication matrix saved as celltype_communication.txt. Then re-run the GRN inference with GENIE3 and GRNBoost using their default parameters, writing each numerical adjacency matrix (regulator-target weights, no extra columns) to a *.txt file and feeding it in via python MDIC3.py -exp=... -label=... -grnchoose='other' -grn=GRN.txt -out=... . Compare the resulting communication matrices element-wise (pairwise correlation and per-pair difference magnitudes across all directed cell-type pairs) since the accuracy of the GRN can affect MDIC3 results. Validate that all matrices share identical dimensions and orientation (row i = sender, column j = receiver) before interpreting differences as tool-driven variability.

## Related tools

- **GNIPLR** (Reference GRN inference algorithm producing the baseline adjacency matrix used by MDIC3) — https://github.com/zyllluck/GNIPLR
- **GENIE3** (Alternative GRN inference tool (default parameters) whose adjacency matrix replaces the GNIPLR GRN in MDIC3)
- **GRNBoost** (Alternative GRN inference tool (default parameters) whose adjacency matrix replaces the GNIPLR GRN in MDIC3)
- **MDIC3** (Downstream method that consumes the GRN adjacency matrix as left submatrix R and outputs cell-type communication matrices) — https://github.com/LYxiaotai/MDIC3
- **Python** (Language used to run MDIC3.py/GNIPLR scripts and perform element-wise matrix comparisons)

## Examples

```
python MDIC3.py -exp=scRNA_expression_file -label=cell_label_file -grnchoose='other' -grn=GRN.txt -out=results_output_fold
```

## Evaluation signals

- All three communication matrices have identical dimensions and orientation: row i = sender cell type, column j = receiver cell type, matching the celltype_communication.txt convention
- The comparison table contains communication strengths for all 49 directed cell-type pairs (7x7) under each of the three GRN tools
- The imported GRN *.txt file contains only numerical adjacency matrix values, as required by the -grnchoose='other' pathway
- Pairwise correlations between the three communication matrices are computed and per-pair difference magnitudes reported; large divergences indicate GRN-tool sensitivity
- The GNIPLR run used explicit step and process parameters (e.g., step=500, process=3) so the baseline is reproducible

## Limitations

- No software version numbers were stated for any tool (MDIC3, GNIPLR, GENIE3, GRNBoost), limiting exact reproducibility
- The accuracy of the GRN directly affects MDIC3's cell-cell communication results, so substitution effects reflect GRN quality rather than MDIC3 behavior
- The paper used the alternative tools only with their default parameters and provides no guidance on tuning GENIE3/GRNBoost for specific datasets

## Evidence

- [discussion] We used two regulatory network inference tools (GENIE3118 and GRNBoost119) to replace the GRN from GNIPLR: "We used two regulatory network inference tools (GENIE3118 and GRNBoost119) to replace the GRN from GNIPLR"
- [discussion] GRN accuracy affects results: "the accuracy of the GRN can affect the results of cell-cell communication for MDIC3"
- [readme] README import-other-GRN pathway: "users can also choose to import the GRN calculated by other methods or tools and then infer cell-cell communications using MDIC3"
- [methods] MDIC3 core inference principle: "infer the regulatory relationships among cells based on the regulatory relationships among genes"
