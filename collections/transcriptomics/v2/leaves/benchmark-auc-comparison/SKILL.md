---
name: benchmark-auc-comparison
description: Use when you need to quantitatively compare cell-cell communication inference
  methods (e.g., MDIC3 vs CellChat) on the same scRNA-seq dataset using literature-supported
  ground truth and AUC, rather than relying on qualitative claims.
when_to_use_negative:
- You lack a curated literature-supported ground truth for the dataset — without it
  the AUC cannot be computed and the comparison degenerates to counting communications.
- You want to compare methods on a species whose L-R pairs are unknown — the comparison
  is only meaningful if at least one method (e.g., CellChat) has an applicable L-R
  database; MDIC3's advantage is precisely on such species.
- Your goal is to validate single-cell (individual cell) communication networks —
  the article notes these are difficult to verify, so benchmarking should be restricted
  to cell-type-level communications.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MDIC3
  role: Unsupervised method under benchmark; infers cell-cell communication via matrix
    decomposition with a GRN from scRNA-seq data
  repo: https://github.com/LYxiaotai/MDIC3
- name: CellChat
  role: L-R coexpression-based baseline; provides CellChatDB ligand-receptor database
    used across methods and benchmarking communications counts
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: L-R coexpression-based baseline method included in the default-parameters
    comparison
- name: iTALK
  role: L-R coexpression-based baseline method included in the default-parameters
    comparison
- name: GNIPLR
  role: Infers the gene regulatory network (GRN) that MDIC3 uses as its left submatrix
    R before communication inference and benchmarking
  repo: https://github.com/zyllluck/GNIPLR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:4e203fab706bccc5929f87f856b11542ee196a3b030479e455b24272c084311c
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
  - pheatmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# benchmark-auc-comparison

## Summary

Benchmarks MDIC3's inferred cell-cell communications against L-R coexpression methods (CellChat, CellPhoneDB, iTALK) on the same scRNA-seq dataset, scoring each method with the AUC of a dichotomous classification against literature-supported ground-truth interactions and comparing the number of inferred communications.

## When to use

Use when you need to quantitatively compare cell-cell communication inference methods (e.g., MDIC3 vs CellChat) on the same scRNA-seq dataset using literature-supported ground truth and AUC, rather than relying on qualitative claims.

## When NOT to use

- You lack a curated literature-supported ground truth for the dataset — without it the AUC cannot be computed and the comparison degenerates to counting communications.
- You want to compare methods on a species whose L-R pairs are unknown — the comparison is only meaningful if at least one method (e.g., CellChat) has an applicable L-R database; MDIC3's advantage is precisely on such species.
- Your goal is to validate single-cell (individual cell) communication networks — the article notes these are difficult to verify, so benchmarking should be restricted to cell-type-level communications.

## Inputs

- Single-cell gene expression matrix formatted as exp.txt (genes as rows, cells as columns, no header field)
- Cell-type metadata as metadata.txt (column 1 cells, column 2 cell labels)
- Ground-truth set of literature-supported ligand-receptor interactions for the dataset
- CellChatDB mouse ligand-receptor file
- GRN adjacency matrix from GNIPLR (or another GRN tool)
- Cell-type communication matrices from all benchmarked methods (e.g., celltype_communication.txt)

## Outputs

- AUC values per method against literature-supported ground truth (e.g., MDIC3 0.711 vs CellChat 0.38)
- Counts of inferred communications per method (e.g., MDIC3 43 vs CellChat 34 on GSE60361)
- MSE between the reconstructed matrix and the original single-cell gene expression matrix
- Benchmark comparison table/heatmap of inferred communications

## How to apply

Run MDIC3 (via lucky.MDIC3_score with a GNIPLR-inferred GRN) and the comparison methods (CellChat, CellPhoneDB, iTALK) on the identical expression matrix and cell labels, using all methods with their default parameters and the same L-R database (CellChatDB). Compile a ground-truth set of literature-supported interactions for the dataset (e.g., known mouse-brain IN-microglia pairs such as Apoe-Lrp1, Cx3cl1-Cx3cr1, Ptn-Ncam) and frame performance evaluation as a dichotomous classification problem evaluated by AUC. Count the total communications each method infers (for GSE60361, MDIC3 43 vs CellChat 34) and report AUCs side by side (0.711 vs 0.38). As an internal check of the matrix decomposition itself, compute the mean squared error between the reconstructed matrix and the original single-cell gene expression matrix. The rationale is that existing methods rely on L-R coexpression and prior L-R databases, so benchmarking against them isolates MDIC3's unsupervised, species-agnostic inference advantage.

## Related tools

- **MDIC3** (Unsupervised method under benchmark; infers cell-cell communication via matrix decomposition with a GRN from scRNA-seq data) — https://github.com/LYxiaotai/MDIC3
- **CellChat** (L-R coexpression-based baseline; provides CellChatDB ligand-receptor database used across methods and benchmarking communications counts) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (L-R coexpression-based baseline method included in the default-parameters comparison)
- **iTALK** (L-R coexpression-based baseline method included in the default-parameters comparison)
- **GNIPLR** (Infers the gene regulatory network (GRN) that MDIC3 uses as its left submatrix R before communication inference and benchmarking) — https://github.com/zyllluck/GNIPLR

## Examples

```
python MDIC3.py -exp=exp.txt -label=metadata.txt -grnchoose='GNIPLR' -process=process_value -step=GRN_calculation_step -out=results_output_fold
```

## Evaluation signals

- MDIC3-vs-CellChat comparison on GSE60361 yields 43 vs 34 inferred communications
- AUC against literature-supported ground truth reaches 0.711 for MDIC3 vs 0.38 for CellChat
- All benchmarked methods were run with their default parameters and the same L-R database, ensuring a like-for-like comparison
- MSE between the reconstructed matrix and the original single-cell gene expression matrix confirms the matrix decomposition is sound before interpreting benchmark results
- The reproduced heatmap recapitulates IN-microglia bidirectional communication, microglia to S1PN/CA1PN/IN signaling, and S1PN autocrine/S1PN-IN signaling

## Limitations

- No software version numbers are stated for any benchmarked tool (MDIC3, GNIPLR, GENIE3, GRNBoost, CellChat, CellPhoneDB, iTALK), so results may not be exactly reproducible.
- The accuracy of the GRN inferred by GNIPLR can affect the results of cell-cell communication for MDIC3, and hence its benchmark scores.
- Existing L-R databases are incomplete — many L-R pairs remain undiscovered (e.g., orphan receptors DR6/TNFRSF21, RELT, TROY, NGFR) — which biases ground-truth-based evaluation toward known interactions.
- Equation 5-based decomposition is only applicable when the number of cells equals the number of genes and RΣ is invertible; with cells exceeding genes some information may be missing in the right submatrix.

## Evidence

- [discussion] AUC evaluation framing: "the performance evaluation of the communication results became a dichotomous problem and was evaluated using the AUC."
- [discussion] MSE internal check of decomposition: "calculating the mean squared error (MSE) between the reconstructed matrix and the original single-cell gene expression matrix"
- [discussion] Benchmarks rely on L-R coexpression: "In contrast, other methods, such as CellChat, CellPhoneDB, and iTALK, investigate cell-cell communications on the basis of L-R coexpression"
- [abstract] Headline benchmark result: "By comparing it with existing methods for the inference of cell-cell communication, MDIC3 obtained better performance in both humans and mice."
- [discussion] Default-parameter fairness of comparison: "The three methods were used by their default parameters and the L-R database."
