---
name: vdj-chain-productivity-filtering
description: Use when a paired scRNA-seq + scVDJ-seq SingleCellExperiment object (e.g.
  Suo et al. fetal immune atlas with contig data merged via scRepertoire) must be
  filtered to cells with productive, unambiguous immune receptors before VDJ pseudobulking.
when_to_use_negative:
- Input cells already lack paired VDJ contig data or only BCR sequencing is available
  (e.g. Bandyopadhyay et al. / Baccin et al. datasets lack BCR sequencing for B cell
  workflows)
- The dataset contains only a very small number of target cells (e.g. Tonglin et al.
  with 972 healthy B cells), where filtering would leave insufficient material for
  downstream miloR neighbourhood construction
- You need Dandelion's additional filtering or network-generation functionality, which
  dandelionR has not yet implemented
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: Provides setupVdjPseudobulk, which performs the actual filtering of non-productive
    immune receptors and ambiguous VDJ chain status on the SingleCellExperiment
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: Used to load and combine VDJ contigs (loadContigs, combineTCR with removeNA
    = TRUE, removeMulti = FALSE, filterMulti = TRUE) and attach them to the expression
    object via combineExpression before filtering
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation whose filtering behavior the dandelionR preprocessing
    step reproduces; produces 17,234 cells vs dandelionR's 17,281 on Suo et al. data
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:ec8e42d2bbea47384536cd4965e3f7997575534f6709c1f2f40e5034e46e3e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# vdj-chain-productivity-filtering

## Summary

A preprocessing skill from the dandelionR workflow that filters single-cell VDJ-seq data using dandelionR::setupVdjPseudobulk, removing cells with non-productive immune receptors and ambiguous VDJ chain status so only cells with relevant or complete VDJ information proceed to trajectory analysis.

## When to use

Use when a paired scRNA-seq + scVDJ-seq SingleCellExperiment object (e.g. Suo et al. fetal immune atlas with contig data merged via scRepertoire) must be filtered to cells with productive, unambiguous immune receptors before VDJ pseudobulking.

## When NOT to use

- Input cells already lack paired VDJ contig data or only BCR sequencing is available (e.g. Bandyopadhyay et al. / Baccin et al. datasets lack BCR sequencing for B cell workflows)
- The dataset contains only a very small number of target cells (e.g. Tonglin et al. with 972 healthy B cells), where filtering would leave insufficient material for downstream miloR neighbourhood construction
- You need Dandelion's additional filtering or network-generation functionality, which dandelionR has not yet implemented

## Inputs

- SingleCellExperiment object of scRNA-seq data (e.g. Suo et al. fetal immune atlas Lymphoid Cells GEX from developmental.cellatlas.io)
- Paired scVDJ-seq contig data in AIRR format (e.g. from github.com/zktuong/dandelion-demo-files)
- Filter configuration: already.productive = FALSE and allowed_chain_status = c('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception')
- mode_option (e.g. 'abT') selecting the receptor mode for setupVdjPseudobulk

## Outputs

- Filtered SingleCellExperiment containing only cells with complete/relevant TCR information (17,308 of 65,102 Suo et al. cells retained)
- preprocessing_summary.csv with columns cells_input and cells_retained

## How to apply

Load the gene expression data as a SingleCellExperiment and merge paired scVDJ-seq contig data using scRepertoire (loadContigs, combineTCR with removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE, then combineExpression). Call dandelionR::setupVdjPseudobulk with already.productive = FALSE and allowed_chain_status = c('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception') so non-productive receptors and ambiguous chain statuses are dropped while relevant or complete VDJ data is retained; for the Suo et al. abT data use mode_option = 'abT'. The rationale is that downstream VDJ feature-space construction requires complete TCR information. Record cells_input and cells_retained (for Suo et al., 65102 input, 17308 retained) in a preprocessing_summary.csv and verify the retained count.

## Related tools

- **dandelionR** (Provides setupVdjPseudobulk, which performs the actual filtering of non-productive immune receptors and ambiguous VDJ chain status on the SingleCellExperiment) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (Used to load and combine VDJ contigs (loadContigs, combineTCR with removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE) and attach them to the expression object via combineExpression before filtering) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation whose filtering behavior the dandelionR preprocessing step reproduces; produces 17,234 cells vs dandelionR's 17,281 on Suo et al. data) — https://github.com/zktuong/dandelion

## Examples

```
sce <- dandelionR::setupVdjPseudobulk(sce, mode_option = "abT", already.productive = FALSE, allowed_chain_status = c("Single pair", "Orphan VDJ", "Orphan VDJ-exception"))
```

## Evaluation signals

- Retained cell count matches the reported value: 17,308 cells retained out of 65,102 in the Suo et al. data
- No cells in the filtered SingleCellExperiment have non-productive immune receptors or an ambiguous VDJ chain status outside the allowed set ('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception')
- Cross-tool check: dandelionR output (17,281 cells) vs original Dandelion output (17,234 cells) share 17,208 cells
- preprocessing_summary.csv contains cells_input = 65102 and cells_retained = 17308 with correct schema

## Limitations

- dandelionR has not yet been applied to B cell development data, and cyclic processes such as B cell receptor editing cannot be represented by the absorbing Markov chain downstream
- Future dandelionR work is needed to implement additional filtering and network-generation functionality from the original Dandelion Python package
- R and Python differ in filtering granularity: dandelionR output contained 17,281 cells versus 17,234 from original Dandelion (17,208 shared), so exact counts are tool-dependent

## Evidence

- [methods] Preprocessing (filtering non-productive and ambiguous VDJ chains): "This step includes filtering cells with non-productive immune receptors and ambiguous VDJ chain status"
- [methods] setupVdjPseudobulk filter step: "sce <- setupVdjPseudobulk(sce,
    mode_option = "abT","
- [results] Retained cell count on Suo et al. data: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [results] R vs Python output comparison: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [methods] scRepertoire contig combination: "combined.TCR <- combineTCR(contig.list,
    removeNA = TRUE,
    removeMulti = FALSE,
    filterMulti = TRUE"
- [readme] dandelionR role: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire)"
