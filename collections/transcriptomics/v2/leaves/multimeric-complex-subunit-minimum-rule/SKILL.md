---
name: multimeric-complex-subunit-minimum-rule
description: Use when a curated ligand-receptor pair list annotates protein complexes
  (e.g. Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs) and per-pair
  ligand/receptor scores must be computed from single-cell expression of individual
  subunits.
when_to_use_negative:
- The LR pair list does not annotate protein complexes (e.g. Ramilowski et al. 2015
  or Qiao et al. 2014 lists), so no subunit-level minimum is possible
- Communication scoring is done at the signaling-pathway level (e.g. CellChat pathway-level
  scores) rather than at individual LR-pair level
tools:
- name: cell2cell
  role: cell2cell.preprocessing computes sender ligand and receiver receptor scores
    with the minimum-across-subunits rule for heteromeric complexes and aggregates
    single-cell scores to cell-type level
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Source of the curated LR pair list (Jin et al. 2020/2021) that annotates multi-subunit
    ligand-receptor complexes
  repo: https://github.com/sqjin/CellChat
- name: Tensor-cell2cell
  role: Consumes the subunit-collapsed communication scores assembled into the 4D
    communication tensor for context-aware decomposition
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:34576abad01f1f4d4b2e536a93adfc29de0ba53558b13c6a4cc80ff7f1c9bc9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# multimeric-complex-subunit-minimum-rule

## Summary

A scoring convention for heteromeric ligand or receptor complexes in ligand-receptor communication analysis: when a curated LR pair (e.g. from CellChat's Human-2020-Jin list) contains multi-subunit proteins, the complex-level score is taken as the minimum expression-derived score across all subunits before sender/receiver communication scores are computed.

## When to use

Use when a curated ligand-receptor pair list annotates protein complexes (e.g. Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs) and per-pair ligand/receptor scores must be computed from single-cell expression of individual subunits.

## When NOT to use

- The LR pair list does not annotate protein complexes (e.g. Ramilowski et al. 2015 or Qiao et al. 2014 lists), so no subunit-level minimum is possible
- Communication scoring is done at the signaling-pathway level (e.g. CellChat pathway-level scores) rather than at individual LR-pair level

## Inputs

- Curated ligand-receptor pair CSV with complex/subunit annotations (e.g. Human-2020-Jin-LR-pairs.csv)
- Single-cell expression matrix per context/sample
- Cell-type annotations per cell

## Outputs

- Per-LR-pair, per-cell-type-pair communication score matrices with complex scores collapsed to the minimum subunit score
- 3D communication tensors (LR pairs x senders x receivers) stacked into a serialized 4D communication tensor (communication_tensor.pkl)

## How to apply

For each LR pair in the curated list, identify whether the ligand or receptor is heteromeric (multi-subunit complex). Compute the expression-derived score for each individual subunit in the sender (ligand) and receiver (receptor) cell population, then take the minimum across subunits so the complex score reflects its least-expressed, limiting component. Aggregate these single-cell scores to cell-type level with the non-zero fraction strategy implemented in cell2cell.preprocessing, producing the per-LR-pair sender/receiver communication matrices that feed the 3D tensors stacked into the 4D communication tensor (contexts x LR pairs x sender cells x receiver cells). The minimum rule ensures a complex is only considered active when every subunit is expressed, which is why LR lists annotated with protein complexes (e.g. CellChat/Jin et al. 2020 with 2,005 human LR pairs) are preferred.

## Related tools

- **cell2cell** (cell2cell.preprocessing computes sender ligand and receiver receptor scores with the minimum-across-subunits rule for heteromeric complexes and aggregates single-cell scores to cell-type level) — https://github.com/earmingol/cell2cell
- **CellChat** (Source of the curated LR pair list (Jin et al. 2020/2021) that annotates multi-subunit ligand-receptor complexes) — https://github.com/sqjin/CellChat
- **Tensor-cell2cell** (Consumes the subunit-collapsed communication scores assembled into the 4D communication tensor for context-aware decomposition) — https://github.com/earmingol/cell2cell

## Evaluation signals

- For every heteromeric LR pair, the complex score must never exceed the score of its lowest-expressed subunit (min-rule invariant)
- Monomeric LR pairs must be unchanged by the rule (score equals the single subunit's score)
- Resulting 4D tensor must have dimensions corresponding to contexts, LR pairs, sender cell types, and receiver cell types
- Reloading communication_tensor.pkl must reproduce the original communication scores
- Scores are aggregated to cell-type level using the non-zero fraction strategy, so no cell-type axis retains single-cell granularity

## Limitations

- Only cell types shared across all samples were included in the COVID-19 and ASD case-study tensors, so rare cell types and their complex-mediated interactions may be missed
- The minimum rule discards information about correlations among subunit expression within a complex
- Benchmarking CCC predictions is difficult due to the lack of a ground truth, so the effect of subunit handling on downstream factors cannot be directly validated against truth

## Evidence

- [other] compute sender ligand and receiver receptor scores, taking the minimum across subunits for heteromeric ligands or receptors: [evidence span withheld — anchor status: not_found]
- [results] we use a list of LR pairs published with CellChat: "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [readme] CellChatDB curated database with multi-subunit structure: [evidence span withheld — anchor status: external_doc]
- [methods] cell2cell.preprocessing functions: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
