---
name: ligand-receptor-pair-mapping
description: Use when MDIC3 has produced celltype_communication.txt / cellular_communication.txt
  outputs and you need the specific L-R pairs (e.g., Cx3cl1-Cx3cr1, Apoe-Lrp1, CCL2-CCR2,
  SST-SSTR2) supporting each sender-receiver communication.
when_to_use_negative:
- As the primary inference step when the species lacks curated L-R annotations — MDIC3's
  communication inference is independent of prior L-R knowledge and should run first;
  L-R mapping is only interpretive.
- When the goal is to detect communications involving undiscovered or orphan interactions
  (e.g., orphan receptors DR6/TNFRSF21, RELT, TROY, NGFR) — these are absent from
  L-R databases and cannot be recovered by L-R mapping.
- When no MDIC3 communication output exists — this skill interprets MDIC3 results
  and is not a standalone communication-inference method.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Produces the cell-cell communication scores (celltype_communication.txt, cellular_communication.txt)
    whose underlying L-R pairs are extracted by MDIC3_LR.py
  repo: https://github.com/LYxiaotai/MDIC3
- name: MDIC3_LR.py
  role: Script that extracts L-R pairs from MDIC3-inferred communications, applying
    Bonferroni correction and keeping corrected p < 0.01
  repo: https://github.com/LYxiaotai/MDIC3
- name: CellChatDB
  role: Source of human and mouse ligand-receptor gene pairs (Human_LR.txt / mouse
    L-R file) used as the L-R database
  repo: https://github.com/sqjin/CellChat
- name: GNIPLR
  role: Infers the gene regulatory network upstream of MDIC3, whose output communications
    are annotated with L-R pairs
  repo: https://github.com/zyllluck/GNIPLR
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:7793012f5b0f250945304a425119d65b342f5896e4eaaf21be7639a062144fe6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - CellChatDB
  - CellPhoneDB
  - GNIPLR
  - MDIC3
  - MDIC3_LR.py
  - Python
  - iTALK
  - pheatmap
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# ligand-receptor-pair-mapping

## Summary

Post-hoc extraction of ligand-receptor pairs underlying cell-cell communications inferred by MDIC3, using the CellChatDB-derived L-R database and Bonferroni-corrected significance testing (corrected p < 0.01) to interpret matrix-decomposition communication scores.

## When to use

Use when MDIC3 has produced celltype_communication.txt / cellular_communication.txt outputs and you need the specific L-R pairs (e.g., Cx3cl1-Cx3cr1, Apoe-Lrp1, CCL2-CCR2, SST-SSTR2) supporting each sender-receiver communication.

## When NOT to use

- As the primary inference step when the species lacks curated L-R annotations — MDIC3's communication inference is independent of prior L-R knowledge and should run first; L-R mapping is only interpretive.
- When the goal is to detect communications involving undiscovered or orphan interactions (e.g., orphan receptors DR6/TNFRSF21, RELT, TROY, NGFR) — these are absent from L-R databases and cannot be recovered by L-R mapping.
- When no MDIC3 communication output exists — this skill interprets MDIC3 results and is not a standalone communication-inference method.

## Inputs

- celltype_communication.txt (MDIC3 sender-by-receiver cell-type communication adjacency)
- cellular_communication.txt (MDIC3 single-cell communication adjacency)
- Human_LR.txt or mouse L-R database derived from CellChatDB
- Formatted scRNA-seq exp.txt (genes as rows, cells as columns, no header field) and metadata.txt (cells, cell labels)

## Outputs

- Filtered list of significant ligand-receptor pairs per sender-receiver cell-type pair (corrected p < 0.01)
- L-R pair assignments supporting inferred communications, e.g., CCL2-CCR2 for CM4-to-CM2/CM3, Cx3cl1-Cx3cr1 and Apoe-Lrp1 for interneuron-microglia signaling

## How to apply

After running MDIC3 (GNIPLR GRN inference followed by lucky.MDIC3_score and lucky.MDIC3_scoresave), load the CellChatDB-derived Human_LR.txt or mouse L-R database with lucky.readLRDB, then compute L-R expression with lucky.LR_exp and run MDIC3_LR.py on the target sender-receiver cell-type pairs. Filter candidate pairs by Bonferroni correction, keeping only pairs with corrected p < 0.01; additionally remove universally expressed genes (expressed in more than half of cell types) and genes with zero expression in more than 70% of cells in a cell type. This step is deliberately downstream of MDIC3's unsupervised inference so that L-R interpretation does not constrain which communications are detected — e.g., SST-SSTR2 delta-to-alpha signaling is recovered by MDIC3 but missed by L-R-based methods like CellChat.

## Related tools

- **MDIC3** (Produces the cell-cell communication scores (celltype_communication.txt, cellular_communication.txt) whose underlying L-R pairs are extracted by MDIC3_LR.py) — https://github.com/LYxiaotai/MDIC3
- **MDIC3_LR.py** (Script that extracts L-R pairs from MDIC3-inferred communications, applying Bonferroni correction and keeping corrected p < 0.01) — https://github.com/LYxiaotai/MDIC3
- **CellChatDB** (Source of human and mouse ligand-receptor gene pairs (Human_LR.txt / mouse L-R file) used as the L-R database) — https://github.com/sqjin/CellChat
- **GNIPLR** (Infers the gene regulatory network upstream of MDIC3, whose output communications are annotated with L-R pairs) — https://github.com/zyllluck/GNIPLR

## Examples

```
L, R, LR = lucky.readLRDB('Human_LR.txt'); LRexp = lucky.LR_exp(LR, L, R, gene_exp, label_cell, cellname)
```

## Evaluation signals

- The filtered L-R list contains the expected literature-supported pairs with Bonferroni-corrected p < 0.01 (e.g., CCL2-CCR2, CX3CL1-CX3CR1, CD40-CD40LG, CXCL10-CXCR3 in the LN dataset; Apoe-Lrp1 and Ptn-Ngc in the mouse brain).
- SST-SSTR2 signaling (delta-to-alpha in human islets) appears in the MDIC3 L-R extraction but not in the outputs of L-R-based methods (CellChat, CellPhoneDB, iTALK).
- Genes removed as universally expressed (significant in > half of cell types) or zero-expressed in > 70% of cells within a cell type are excluded from candidate pairs.
- Extracted pairs map onto the significant (red) sender-receiver positions in the pheatmap visualization of celltype_communication.txt (blue-white-red ramp, no clustering).
- Benchmark support rate of extracted L-R pairs matches reported values, e.g., 83.33% literature support for 12 MDIC3 communications in the human islet dataset (GSE116753).

## Limitations

- Existing L-R databases are incomplete — many pairs remain undiscovered, including receptors lacking known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and ligands without receptors (IL17D), so L-R mapping cannot annotate all MDIC3-inferred communications.
- The accuracy of the underlying GRN from GNIPLR affects MDIC3 communication results and hence which L-R pairs are recovered.
- CellChatDB covers only human and mouse L-R information, constraining L-R interpretation to those species even though MDIC3 itself is species-agnostic.

## Evidence

- [discussion] The Bonferroni correction method was used to adjust the p values... The L-R pairs with corrected p values less than 0.01 were considered significant.: [evidence span withheld — anchor status: not_found]
- [discussion] Considering that the CellChatDB L-R database contains both human and mouse L-R information, we obtained human and mouse L-R genes from CellChatDB.: "Considering that the CellChatDB L-R database contains both human and mouse L-R information, we obtained human and mouse L-R genes from CellChatDB."
- [discussion] if a gene is significantly expressed in more than half of the cell types, we consider it a universally expressed gene: "if a gene is significantly expressed in more than half of the cell types, we consider it a universally expressed gene"
- [methods] L, R, LR = lucky.readLRDB('Human_LR.txt'): "L, R, LR = lucky.readLRDB('Human_LR.txt')"
- [methods] we use a simple method to extract L-R pairs from cell-cell communication: "we use a simple method to extract L-R pairs from cell-cell communication"
- [full_text] Extract L-R pairs underlying the inferred communications... using MDIC3_LR.py with the mouse CellChatDB ligand-receptor file, applying Bonferroni correction and keeping pairs with corrected p < 0.01.: [evidence span withheld — anchor status: not_found]
