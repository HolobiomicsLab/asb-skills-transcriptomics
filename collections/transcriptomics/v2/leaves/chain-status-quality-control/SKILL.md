---
name: chain-status-quality-control
description: Use when a SingleCellExperiment object merged with scVDJ-seq contig data
  contains non-productive receptors or ambiguous VDJ chain annotations that must be
  removed before VDJ feature-space or pseudobulk analyses.
when_to_use_negative:
- The input is already a filtered, feature-space object downstream of setupVdjPseudobulk
  (re-running would discard valid cells).
- 'Analysing BCR data with T-cell-oriented settings: the article used mode_option
  = ''abT''; dandelionR has not yet been applied to B cell development data.'
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides setupVdjPseudobulk, which filters non-productive receptors and ambiguous
    VDJ chain statuses and sets up pseudobulk VDJ analysis
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: Loads contigs (loadContigs), combines TCR pairs (combineTCR) and attaches
    them to the SingleCellExperiment (combineExpression) prior to filtering
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation whose filtering behaviour dandelionR reproduces;
    used to cross-check cell retention (17,234 vs 17,281 cells, 17,208 shared)
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:b9e4e7bee344ffe02154488b562f92a7514c8dde09665772715dd41a8896d143
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chain-status-quality-control

## Summary

Quality-control step that filters single-cell VDJ data by immune-receptor productivity and chain status before downstream trajectory analyses, retaining only cells with relevant or complete VDJ information (e.g. 17,308 of 65,102 Suo et al. cells retained).

## When to use

Use when a SingleCellExperiment object merged with scVDJ-seq contig data contains non-productive receptors or ambiguous VDJ chain annotations that must be removed before VDJ feature-space or pseudobulk analyses.

## When NOT to use

- The input is already a filtered, feature-space object downstream of setupVdjPseudobulk (re-running would discard valid cells).
- Analysing BCR data with T-cell-oriented settings: the article used mode_option = 'abT'; dandelionR has not yet been applied to B cell development data.

## Inputs

- SingleCellExperiment object with gene expression (e.g. fetal immune atlas Lymphoid Cells GEX)
- Paired single-cell VDJ-seq contig data in AIRR format (e.g. Suo et al. single-cell VDJ data)
- scRepertoire combined TCR contig list

## Outputs

- Filtered SingleCellExperiment retaining only cells with complete/relevant VDJ data (e.g. 17,308 cells)
- Summary table of cells_input and cells_retained (e.g. preprocessing_summary.csv)

## How to apply

Merge the gene-expression data (e.g. the Suo et al. fetal-immune atlas as a SingleCellExperiment) with paired scVDJ-seq contigs using scRepertoire's loadContigs, combineTCR (removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE) and combineExpression. Then run dandelionR::setupVdjPseudobulk with already.productive = FALSE and allowed_chain_status = c('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception') so that cells with non-productive immune receptors and ambiguous VDJ chain status are filtered out, retaining cells with complete TCR information. Record cells_input and cells_retained (e.g. 65,102 in, 17,308 retained for the abT mode on Suo et al.) and compare to the reported count to confirm reproducibility. This step precedes pseudobulking (miloR) and VDJ feature-space construction in the dandelionR workflow.

## Related tools

- **dandelionR** (Provides setupVdjPseudobulk, which filters non-productive receptors and ambiguous VDJ chain statuses and sets up pseudobulk VDJ analysis) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (Loads contigs (loadContigs), combines TCR pairs (combineTCR) and attaches them to the SingleCellExperiment (combineExpression) prior to filtering) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation whose filtering behaviour dandelionR reproduces; used to cross-check cell retention (17,234 vs 17,281 cells, 17,208 shared)) — https://github.com/zktuong/dandelion

## Examples

```
sce <- setupVdjPseudobulk(sce, mode_option = "abT", already.productive = FALSE, allowed_chain_status = c("Single pair", "Extra pair", "Extra pair-exception", "Orphan VDJ", "Orphan VDJ-exception"))
```

## Evaluation signals

- Retained cell count matches the reported 17,308 of 65,102 input cells after setupVdjPseudobulk on the Suo et al. abT data.
- No cells with non-productive receptors or disallowed chain statuses (outside 'Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception') remain.
- Summary table contains cells_input and cells_retained columns consistent with the filtered object's colData dimensions.
- Cross-implementation check: dandelionR output (17,281 cells) shares 17,208 cells with original Dandelion output (17,234 cells).

## Limitations

- Filter settings are T-cell (abT mode) oriented; dandelionR has not been applied to B cell development data, which involves more complex BCR processes.
- dandelionR's retained set differs slightly from original Dandelion (17,281 vs 17,234 cells), due to discrepancies in diffusion map and KNN graph construction between R and Python packages.
- Future work is needed to implement additional filtering and network generation functionality from the original Dandelion Python package.

## Evidence

- [methods] Filtering non-productive and ambiguous VDJ chains via setupVdjPseudobulk parameters: "This step includes filtering cells with non-productive immune receptors and ambiguous VDJ chain status"
- [results] Cell retention counts for Suo et al. data: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [results] Cross-implementation comparison of filtered outputs: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [methods] setupVdjPseudobulk call in the workflow: "sce <- setupVdjPseudobulk(sce,
    mode_option = "abT","
- [intro] dandelionR scope statement: [evidence span withheld — anchor status: not_found]
