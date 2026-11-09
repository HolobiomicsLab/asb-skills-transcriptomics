---
name: single-cell-vdj-preprocessing
description: Use when raw scVDJ-seq contigs (e.g. AIRR/contig formats) must be merged
  with a SingleCellExperiment GEX object and cells without complete, productive TCR
  information filtered out before pseudobulking or trajectory analysis.
when_to_use_negative:
- The dataset lacks paired single-cell VDJ/immune-receptor sequencing (e.g. Baccin
  et al. / Baccin-type bone marrow data with no BCR sequencing) — filtering on VDJ
  chain status is impossible or meaningless.
- Input has already been through setupVdjPseudobulk or equivalent Dandelion filtering
  and only contains productive, unambiguous VDJ chains.
- The goal is trajectory analysis on cyclic processes (e.g. B cell receptor editing)
  that cannot be represented by absorbing Markov chains — preprocessing alone does
  not address the downstream topology limitation.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: R package whose setupVdjPseudobulk function filters non-productive receptors
    and ambiguous VDJ chain status and enables downstream pseudobulk VDJ feature-space
    construction
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: Used to load AIRR-format contigs (loadContigs), combine TCR chains (combineTCR
    with removeNA=TRUE, removeMulti=FALSE, filterMulti=TRUE), and attach receptor
    data to the SCE (combineExpression)
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation of the VDJ-feature workflow; setupVdjPseudobulk
    reproduces its filtering behavior and its output (17,234 cells) serves as a cross-package
    benchmark
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:4d61db3f1c3c5b2089fa0f747780415c1db6939a86a573b46cfc4ca76f3b2173
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-vdj-preprocessing

## Summary

Preprocess paired single-cell VDJ-seq and gene-expression data by filtering non-productive immune receptors and ambiguous VDJ chain status (via dandelionR::setupVdjPseudobulk), yielding a filtered SingleCellExperiment with complete TCR information for downstream VDJ-feature-space trajectory analysis.

## When to use

Use when raw scVDJ-seq contigs (e.g. AIRR/contig formats) must be merged with a SingleCellExperiment GEX object and cells without complete, productive TCR information filtered out before pseudobulking or trajectory analysis.

## When NOT to use

- The dataset lacks paired single-cell VDJ/immune-receptor sequencing (e.g. Baccin et al. / Baccin-type bone marrow data with no BCR sequencing) — filtering on VDJ chain status is impossible or meaningless.
- Input has already been through setupVdjPseudobulk or equivalent Dandelion filtering and only contains productive, unambiguous VDJ chains.
- The goal is trajectory analysis on cyclic processes (e.g. B cell receptor editing) that cannot be represented by absorbing Markov chains — preprocessing alone does not address the downstream topology limitation.

## Inputs

- SingleCellExperiment object with gene expression data (e.g. Suo et al. fetal immune atlas Lymphoid Cells GEX)
- Paired single-cell VDJ-seq contig data in AIRR format (scRepertoire contig list)
- Filtering parameters: already.productive=FALSE, allowed_chain_status list, mode_option lineage (e.g. 'abT')

## Outputs

- Filtered SingleCellExperiment containing only cells with complete TCR information
- preprocessing_summary.csv with cells_input and cells_retained counts
- Combined TCR contig list from combineTCR for downstream scRepertoire/dandelionR steps

## How to apply

Load the fetal-immune-style gene expression data as a SingleCellExperiment object and merge it with paired scVDJ-seq contig data using scRepertoire's loadContigs (format='AIRR'), combineTCR (removeNA=TRUE, removeMulti=FALSE, filterMulti=TRUE), and combineExpression. Then run dandelionR::setupVdjPseudobulk with already.productive=FALSE and allowed_chain_status=c('Single pair','Extra pair','Extra pair-exception','Orphan VDJ','Orphan VDJ-exception'), selecting a mode such as mode_option='abT' to restrict to the lineage of interest; this step removes cells with non-productive immune receptors and ambiguous VDJ chain status. The rationale is that downstream VDJ feature-space construction requires complete receptor information — in the Suo et al. fetal-immune atlas, this filtering retained 17,308 of 65,102 cells. Record cells_input and cells_retained (e.g. to preprocessing_summary.csv) and compare the retained count against reported benchmarks (dandelionR output: 17,281 cells; original Dandelion: 17,234 cells; 17,208 shared).

## Related tools

- **dandelionR** (R package whose setupVdjPseudobulk function filters non-productive receptors and ambiguous VDJ chain status and enables downstream pseudobulk VDJ feature-space construction) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (Used to load AIRR-format contigs (loadContigs), combine TCR chains (combineTCR with removeNA=TRUE, removeMulti=FALSE, filterMulti=TRUE), and attach receptor data to the SCE (combineExpression)) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation of the VDJ-feature workflow; setupVdjPseudobulk reproduces its filtering behavior and its output (17,234 cells) serves as a cross-package benchmark) — https://github.com/zktuong/dandelion

## Examples

```
sce <- setupVdjPseudobulk(sce, mode_option = "abT", already.productive = FALSE, allowed_chain_status = c("Single pair", "Extra pair", "Extra pair-exception", "Orphan VDJ", "Orphan VDJ-exception"))
```

## Evaluation signals

- The retained cell count (17,308 from 65,102 Suo et al. cells) matches the published figure for the same filtering parameters.
- Cross-check against the original Dandelion pipeline: dandelionR yields 17,281 cells vs. 17,234 in Dandelion, with 17,208 cells shared — large divergence signals parameter or data-handling mismatch.
- Every retained cell has a complete, productive TCR chain with an allowed chain status ('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception'); no non-productive or ambiguous-status cells remain.
- preprocessing_summary.csv contains both cells_input and cells_retained columns and cells_retained <= cells_input.
- combineTCR was called with removeNA=TRUE, removeMulti=FALSE, filterMulti=TRUE so cells lacking TCR contigs are excluded before combineExpression.

## Limitations

- Heavy cell loss is expected: filtering retained only 17,308 of 65,102 cells, so datasets with sparse or incomplete VDJ information may retain very few cells.
- The destiny-based diffusion map downstream limits trajectory inference to bifurcation or linear topologies, so preprocessed output is not suitable for cyclic trajectories such as B cell receptor editing.
- dandelionR has not yet been applied to B cell development data, which involves more complex receptor configurations than abT-mode T cell data.
- Small datasets (e.g. 972 healthy B cells in Tonglin et al.) may be insufficient even for downstream miloR neighbourhood construction after filtering.

## Evidence

- [methods] Filtering of non-productive/ambiguous VDJ status via setupVdjPseudobulk: "This step includes filtering cells with non-productive immune receptors and ambiguous VDJ chain status"
- [results] Retention counts on Suo et al. data: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [results] Cross-package benchmark: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [results] setupVdjPseudobulk applied to Suo et al. VDJ data: "The dandelionR::setupVdjPseudobulk function preprocessed the single-cell VDJ data by Suo et al. [24]."
- [readme] README: scRepertoire interface: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
