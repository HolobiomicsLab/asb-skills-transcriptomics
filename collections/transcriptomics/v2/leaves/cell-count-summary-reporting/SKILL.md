---
name: cell-count-summary-reporting
description: Use when a VDJ preprocessing or filtering step (e.g. setupVdjPseudobulk)
  has just been applied and you must quantify and report how many cells were retained
  versus input, as a baseline check before downstream pseudobulking or trajectory
  analysis.
when_to_use_negative:
- Input is a pseudobulk or feature-space object (post-vdjPseudobulk); per-cell filtering
  counts no longer apply
- The dataset has already been quality-filtered upstream, so an input-vs-retained
  count would double-report the same filter step
- Data contains no VDJ information (no paired BCR/TCR contigs), since setupVdjPseudobulk
  filtering is meaningless there
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: Provides setupVdjPseudobulk, the filtering function whose input/retained cell
    counts are recorded
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: loadContigs/combineTCR/combineExpression build the per-cell object whose cell
    counts are summarised
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Python reference implementation whose cell output (17234) is a benchmark against
    which the retained count is compared
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:a460246fd748718e98101d38b6955acdaeccf1db373c96a7e0a0c89ba8c6bfbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-count-summary-reporting

## Summary

Records input and retained cell counts around a filtering/preprocessing step (here, dandelionR::setupVdjPseudobulk on the Suo et al. fetal-immune scVDJ-seq data, 65102 -> 17308 cells) so that cell loss from non-productive or ambiguous VDJ filtering is explicitly documented and auditable.

## When to use

Use when a VDJ preprocessing or filtering step (e.g. setupVdjPseudobulk) has just been applied and you must quantify and report how many cells were retained versus input, as a baseline check before downstream pseudobulking or trajectory analysis.

## When NOT to use

- Input is a pseudobulk or feature-space object (post-vdjPseudobulk); per-cell filtering counts no longer apply
- The dataset has already been quality-filtered upstream, so an input-vs-retained count would double-report the same filter step
- Data contains no VDJ information (no paired BCR/TCR contigs), since setupVdjPseudobulk filtering is meaningless there

## Inputs

- SingleCellExperiment object with merged scRNA-seq and scVDJ-seq data (e.g. Suo et al. fetal-immune Lymphoid Cells GEX plus paired TCR contigs)
- Filtering parameters for setupVdjPseudobulk (already.productive, allowed_chain_status, mode_option)

## Outputs

- Retained cell count (integer) alongside input cell count, e.g. 17308 of 65102
- preprocessing_summary.csv with cells_input and cells_retained columns

## How to apply

Start from the SingleCellExperiment object containing the merged gene-expression and paired scVDJ-seq contig data (built via scRepertoire's loadContigs/combineTCR/combineExpression) and record the starting cell count (65102 for the Suo et al. data). Run dandelionR::setupVdjPseudobulk with already.productive = FALSE and allowed_chain_status = c('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception') so only cells with complete, relevant TCR information are kept. Count the retained cells (expected 17308) and store cells_input and cells_retained in a summary table such as preprocessing_summary.csv. Sanity-check the retained count against the article's reported value and against the closely related cross-package outputs (dandelionR 17281 cells vs original Dandelion 17234 cells, 17208 shared) to confirm the filtering logic is behaving consistently.

## Related tools

- **dandelionR** (Provides setupVdjPseudobulk, the filtering function whose input/retained cell counts are recorded) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (loadContigs/combineTCR/combineExpression build the per-cell object whose cell counts are summarised) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Python reference implementation whose cell output (17234) is a benchmark against which the retained count is compared) — https://github.com/zktuong/dandelion

## Examples

```
sce <- setupVdjPseudobulk(sce, mode_option = "abT", already.productive = FALSE, allowed_chain_status = c("Single pair", "Extra pair", "Extra pair-exception", "Orphan VDJ", "Orphan VDJ-exception")); sum(colData(sce)$kept)
```

## Evaluation signals

- Retained cell count matches the article's reported value of 17308 out of 65102 cells
- preprocessing_summary.csv contains both cells_input and cells_retained and retained <= input
- Cross-package consistency: dandelionR should give ~17281 retained cells and original Dandelion 17234, with ~17208 shared
- Cells removed are those with non-productive immune receptors or ambiguous VDJ chain status, not cells dropped for other reasons

## Limitations

- dandelionR output (17281 cells) differs from original Dandelion (17234 cells) due to fundamental discrepancies in filtering/diffusion-map implementation, so exact count parity across packages is not expected
- The filtering step retains only cells with complete TCR information; heavy cell loss (~73% here) is expected and should not be treated as an error
- Future work will implement additional filtering functionality from the original Dandelion Python package, so current counts may change in later dandelionR versions

## Evidence

- [results] Input/retained cell counts from setupVdjPseudobulk filtering: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [methods] What the filtering step removes, justifying the count drop: "This step includes filtering cells with non-productive immune receptors and ambiguous VDJ chain status"
- [results] Cross-package cell-count comparison used as a verification signal: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [readme] Context of the skill: dandelionR preprocessing workflow: "dandelionR is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation"
