---
name: auc-performance-comparison
description: Use when benchmarking communication-inference tools (e.g., MDIC3 vs CellChat,
  CellPhoneDB, iTALK) against a curated literature-supported set of interactions to
  rank methods by support rate and AUC.
when_to_use_negative:
- When no literature-supported ground-truth set of communications exists for the tissue/species,
  since support percentages and AUC cannot be computed
- When methods were run on different datasets, cell-type label sets, or L-R databases
  — the comparison requires identical inputs across methods
- When the goal is to validate single-cell-level communication networks, which the
  paper notes are difficult to verify
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MDIC3
  role: Method under evaluation; infers communication via GRN and matrix decomposition;
    L-R pairs extracted with MDIC3_LR.py
  repo: https://github.com/LYxiaotai/MDIC3
- name: CellChat
  role: L-R coexpression comparison method run with default parameters and default
    L-R database on the same dataset
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: L-R coexpression comparison method run with default parameters on the same
    dataset
- name: iTALK
  role: L-R coexpression comparison method run with default parameters on the same
    dataset
- name: GNIPLR
  role: Infers the GRN used by MDIC3 (block-wise calculation, step size <= 3000) prior
    to benchmarking
  repo: https://github.com/zyllluck/GNIPLR
- name: Python
  role: Language in which MDIC3 and the benchmark comparison are implemented
  repo: https://github.com/LYxiaotai/MDIC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:d05c28a28ae26d8ab32c12e0cd920f1009713126c2fdb1c2e67d4acd478e4714
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# auc-performance-comparison

## Summary

Evaluates cell-cell communication inference methods by treating their predicted communications as a dichotomous classification problem against literature-supported ground truth, reporting literature-support percentages and AUC values per method.

## When to use

Use when benchmarking communication-inference tools (e.g., MDIC3 vs CellChat, CellPhoneDB, iTALK) against a curated literature-supported set of interactions to rank methods by support rate and AUC.

## When NOT to use

- When no literature-supported ground-truth set of communications exists for the tissue/species, since support percentages and AUC cannot be computed
- When methods were run on different datasets, cell-type label sets, or L-R databases — the comparison requires identical inputs across methods
- When the goal is to validate single-cell-level communication networks, which the paper notes are difficult to verify

## Inputs

- Inferred cell-cell communication matrices from each method (e.g., celltype_communication.txt from MDIC3)
- Extracted L-R pairs per method (e.g., MDIC3_LR.py output with Bonferroni-corrected p < 0.01)
- Literature-supported ground-truth set of cell-cell communications / L-R pairs
- Shared scRNA-seq expression matrix and cell-type labels run identically through all methods (e.g., human islet GSE116753, five endocrine cell types)

## Outputs

- Per-method literature-support percentage
- Per-method AUC from dichotomous classification against literature ground truth
- Benchmark table of communication counts, support percentages, and AUCs (e.g., islet_ccc_benchmark_results.csv)

## How to apply

Run each candidate method (MDIC3 via GNIPLR-inferred GRN; CellChat, CellPhoneDB, iTALK with default parameters and default L-R databases) on the same expression matrix and cell labels, e.g., the human islet dataset (GEO: GSE116753) across five endocrine cell types. For MDIC3, extract L-R pairs with MDIC3_LR.py using the CellChatDB-derived human L-R database and filter candidates at Bonferroni-corrected p < 0.01. Treat each method's inferred communications as predictions and score them against literature-supported ground truth as a dichotomous classification problem, computing the literature-support percentage and the AUC per method. Compile counts, support percentages, and AUCs into a results table (e.g., islet_ccc_benchmark_results.csv) and check reproduction against reported values (counts 12/8/2/7; support 83.33%/12.5%/0%/42.86%; AUC MDIC3 0.801, CellChat 0.247, iTALK 0.449, CellPhoneDB 0.417).

## Related tools

- **MDIC3** (Method under evaluation; infers communication via GRN and matrix decomposition; L-R pairs extracted with MDIC3_LR.py) — https://github.com/LYxiaotai/MDIC3
- **CellChat** (L-R coexpression comparison method run with default parameters and default L-R database on the same dataset) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (L-R coexpression comparison method run with default parameters on the same dataset)
- **iTALK** (L-R coexpression comparison method run with default parameters on the same dataset)
- **GNIPLR** (Infers the GRN used by MDIC3 (block-wise calculation, step size <= 3000) prior to benchmarking) — https://github.com/zyllluck/GNIPLR
- **Python** (Language in which MDIC3 and the benchmark comparison are implemented) — https://github.com/LYxiaotai/MDIC3

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=8 -step=500 -out=results/ ; python MDIC3_LR.py --LRDB Human_LR.txt --p 0.01
```

## Evaluation signals

- Reproduced AUCs match reported values within tolerance: MDIC3 0.801, CellChat 0.247, iTALK 0.449, CellPhoneDB 0.417
- Reproduced communication counts match 12/8/2/7 and support percentages match 83.33%/12.5%/0%/42.86% (MDIC3/CellChat/iTALK/CellPhoneDB)
- SST-SSTR2 signaling (beta-cell autocrine and delta-to-alpha) appears in MDIC3 output but not in the L-R-based methods
- MDIC3 L-R extraction retains only pairs with Bonferroni-corrected p < 0.01
- All methods were run on the same expression matrix, cell labels, and (for L-R methods) default L-R databases

## Limitations

- Ground truth relies on literature-supported L-R pairs, and existing L-R databases are incomplete — many pairs (e.g., orphan receptors DR6, RELT, TROY, NGFR) remain undiscovered, biasing AUC toward L-R-based methods
- The accuracy of the underlying GRN affects MDIC3 results; GRN inference quality is an open dependency
- Communication networks among individual cells are difficult to verify, so evaluation is restricted to cell-type-level communications

## Evidence

- [discussion] AUC evaluation against literature ground truth: "the performance evaluation of the communication results became a dichotomous problem and was evaluated using the AUC."
- [discussion] Default parameters for comparison methods: "The three methods were used by their default parameters and the L-R database."
- [discussion] Bonferroni-corrected significance filter for MDIC3 L-R pairs: [evidence span withheld — anchor status: not_found]
- [readme] MDIC3 output file schema: "'celltype_communication.txt' is the result of communication among different cell types, while the first column and the first row represent different cell labels."
- [discussion] Incomplete L-R databases as evaluation caveat: "there are still many L-R pairs waiting to be discovered, including some receptors lacking known ligands, such as DR6 (TNFRSF21), RELT, TROY, and NGFR"
