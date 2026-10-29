---
name: literature-ground-truth-benchmark-evaluation
description: Use when comparing cell-cell communication inference methods on the same
  scRNA-seq dataset and you need a quantitative, literature-grounded reliability metric
  (support percentage and AUC) for each method's predicted communications.
when_to_use_negative:
- When the goal is to validate single-cell-level communication networks — the paper
  notes communication networks of individual cells are difficult to verify, so this
  evaluation is done at the cell-type level only
- When no literature-supported L-R ground truth exists for the tissue/species, since
  the dichotomous classification relies entirely on literature support
- When you want to assess GRN inference quality itself (e.g., comparing GNIPLR vs
  GENIE3 vs GRNBoost) rather than downstream communication reliability
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MDIC3
  role: Method under evaluation; infers communications via GRN and matrix decomposition,
    and MDIC3_LR.py extracts L-R pairs for literature scoring
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Infers the gene regulatory network that serves as the GRN input (R submatrix)
    for MDIC3 before benchmark scoring
  repo: https://github.com/zyllluck/GNIPLR
- name: CellChat
  role: L-R coexpression-based competitor method run with default parameters and CellChatDB;
    its predictions are scored against literature ground truth
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: L-R coexpression-based competitor method run with default parameters and default
    L-R database for benchmark comparison
- name: iTALK
  role: L-R coexpression-based competitor method run with default parameters and default
    L-R database for benchmark comparison
- name: Python
  role: Implementation environment; MDIC3 and MDIC3_LR.py are Python scripts/packages
    used to generate and extract the communications being scored
  repo: https://github.com/LYxiaotai/MDIC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:548f72c76f177675988e3736fdf3d5edbbc4a756b560685edbd5f49cf3d0e759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - CellPhoneDB
  - GNIPLR
  - MDIC3
  - Python
  - iTALK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# literature-ground-truth-benchmark-evaluation

## Summary

Evaluates inferred cell-cell communications from MDIC3, CellChat, CellPhoneDB, and iTALK against literature-supported ligand-receptor ground truth, treating each method's predictions as a dichotomous classification problem scored by literature-support percentage and AUC. Provides the benchmark logic behind the MDIC3 paper's comparison on the human islet dataset (GSE116753).

## When to use

Use when comparing cell-cell communication inference methods on the same scRNA-seq dataset and you need a quantitative, literature-grounded reliability metric (support percentage and AUC) for each method's predicted communications.

## When NOT to use

- When the goal is to validate single-cell-level communication networks — the paper notes communication networks of individual cells are difficult to verify, so this evaluation is done at the cell-type level only
- When no literature-supported L-R ground truth exists for the tissue/species, since the dichotomous classification relies entirely on literature support
- When you want to assess GRN inference quality itself (e.g., comparing GNIPLR vs GENIE3 vs GRNBoost) rather than downstream communication reliability

## Inputs

- Inferred cell-cell communication predictions from each method (e.g., celltype_communication.txt / cellular_communication.txt from MDIC3; communication results from CellChat, CellPhoneDB, iTALK)
- Extracted L-R pairs per method (MDIC3_LR.py output filtered at Bonferroni-corrected p < 0.01 using CellChatDB human L-R database)
- Human islet scRNA-seq expression matrix (*.txt, rows = genes, columns = cells, GEO: GSE116753) with cell-type metadata (*.txt, cells + labels)
- Literature-supported ground-truth L-R interactions used for dichotomous labeling

## Outputs

- Benchmark results table (islet_ccc_benchmark_results.csv) with communication counts, literature-support percentages, and AUC per method
- Per-method AUC values (MDIC3 0.801, CellChat 0.247, iTALK 0.449, CellPhoneDB 0.417 on the islet benchmark)
- Per-method literature-support percentages (83.33%, 12.5%, 0%, 42.86%) and communication counts (12/8/2/7)
- Method-unique signaling confirmation (e.g., SST-SSTR2 in MDIC3 but absent from L-R-based methods)

## How to apply

Run each inference method on the identical formatted dataset (e.g., the human islet data from GEO: GSE116753 with rows = genes, columns = cells and matched metadata): MDIC3 with a GNIPLR-inferred GRN (block-wise, step <= 3000), and CellChat, CellPhoneDB, and iTALK with their default parameters and default L-R databases. For MDIC3, extract L-R pairs from the inferred communication using MDIC3_LR.py with the CellChatDB-derived human L-R database, filtering candidates with Bonferroni-corrected p < 0.01. Next, treat each method's inferred communications as a dichotomous classification problem by scoring predicted communications against literature-supported interactions, computing the literature-support percentage and the AUC for each method. Compile counts, support percentages, and AUCs into a results table (e.g., islet_ccc_benchmark_results.csv) and validate against reported benchmarks within tolerance; also verify method-unique signaling such as SST-SSTR2 (beta-cell autocrine and delta-to-alpha) appearing only in MDIC3 output.

## Related tools

- **MDIC3** (Method under evaluation; infers communications via GRN and matrix decomposition, and MDIC3_LR.py extracts L-R pairs for literature scoring) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Infers the gene regulatory network that serves as the GRN input (R submatrix) for MDIC3 before benchmark scoring) — https://github.com/zyllluck/GNIPLR
- **CellChat** (L-R coexpression-based competitor method run with default parameters and CellChatDB; its predictions are scored against literature ground truth) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (L-R coexpression-based competitor method run with default parameters and default L-R database for benchmark comparison)
- **iTALK** (L-R coexpression-based competitor method run with default parameters and default L-R database for benchmark comparison)
- **Python** (Implementation environment; MDIC3 and MDIC3_LR.py are Python scripts/packages used to generate and extract the communications being scored) — https://github.com/LYxiaotai/MDIC3

## Examples

```
python MDIC3.py -exp=islet_exp.txt -label=islet_metadata.txt -grnchoose='GNIPLR' -process=8 -step=500 -out=islet_results
```

## Evaluation signals

- Reproduced AUCs match reported benchmarks within tolerance: MDIC3 0.801, CellChat 0.247, iTALK 0.449, CellPhoneDB 0.417
- Communication counts (12/8/2/7) and literature-support percentages (83.33%/12.5%/0%/42.86%) match the benchmark table
- SST-SSTR2 signaling (beta-cell autocrine and delta-to-alpha) appears in MDIC3 output but not in the L-R-based methods' outputs
- All retained L-R pairs for MDIC3 passed the Bonferroni-corrected p < 0.01 significance filter
- All four methods were run on the identical formatted islet dataset (GSE116753) with the same cell-type labels, so counts are directly comparable

## Limitations

- Literature-based ground truth is incomplete: many L-R pairs remain undiscovered, including orphan receptors (DR6/TNFRSF21, RELT, TROY, NGFR) and ligands without known receptors (IL17D), so support percentages underestimate methods that infer beyond known L-R pairs
- GRN quality is an open dependency: the accuracy of the GRN (from GNIPLR or alternatives like GENIE3/GRNBoost) can affect MDIC3's communication results and thus its benchmark score
- Comparison may be biased because CellChat, CellPhoneDB, and iTALK rely on L-R coexpression while MDIC3 is independent of predefined L-R interactions; missing software version numbers for all tools limit reproducibility

## Evidence

- [discussion] Dichotomous AUC evaluation: "the performance evaluation of the communication results became a dichotomous problem and was evaluated using the AUC."
- [discussion] Bonferroni significance filter: "The L-R pairs with corrected p values less than 0.01 were considered significant."
- [discussion] Default-parameter benchmarking: "The three methods were used by their default parameters and the L-R database."
- [readme] Input format requirement: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [discussion] GRN dependency: "the accuracy of the GRN can affect the results of cell-cell communication for MDIC3."
