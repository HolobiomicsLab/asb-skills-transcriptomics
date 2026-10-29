---
name: singlecellexperiment-object-manipulation
description: Use when scVDJ-seq data (gene expression + AIRR/contig receptor data)
  must be combined into a SingleCellExperiment object and its assays, reducedDims,
  or colData manipulated for R-based immune repertoire trajectory analysis.
when_to_use_negative:
- Data are already provided as an SCE with VDJ metadata attached (e.g. the bundled
  sce_vdj object) — skip merging and proceed directly to setupVdjPseudobulk
- The workflow targets Python/AnnData (scanpy) objects — dandelion's Python path,
  not SingleCellExperiment manipulation, is the appropriate route
- The analysis goal is BCR/B-cell trajectory analysis — dandelionR's SCE-based workflow
  has not yet been applied to B cell development data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: Consumes and manipulates SingleCellExperiment objects; setupVdjPseudobulk
    filters cells with non-productive immune receptors and ambiguous VDJ chain status
    on the SCE
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: Provides loadContigs, combineTCR, and combineExpression to merge VDJ contig
    data into the SCE colData
  repo: https://github.com/ncborcherding/scRepertoire
- name: R
  role: Language/environment in which SingleCellExperiment objects are constructed
    and manipulated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:699503a101b55423e454474e08094931a05feb739e0d24c311fe6f61fcd673f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# singlecellexperiment-object-manipulation

## Summary

Manipulate SingleCellExperiment (SCE) objects to carry single-cell gene expression and V(D)J repertoire metadata through the dandelionR preprocessing and pseudobulk workflow, e.g. merging Suo et al. fetal-immune GEX with scRepertoire TCR contig data before running setupVdjPseudobulk.

## When to use

Use when scVDJ-seq data (gene expression + AIRR/contig receptor data) must be combined into a SingleCellExperiment object and its assays, reducedDims, or colData manipulated for R-based immune repertoire trajectory analysis.

## When NOT to use

- Data are already provided as an SCE with VDJ metadata attached (e.g. the bundled sce_vdj object) — skip merging and proceed directly to setupVdjPseudobulk
- The workflow targets Python/AnnData (scanpy) objects — dandelion's Python path, not SingleCellExperiment manipulation, is the appropriate route
- The analysis goal is BCR/B-cell trajectory analysis — dandelionR's SCE-based workflow has not yet been applied to B cell development data

## Inputs

- Gene expression matrix as a SingleCellExperiment object (e.g. Suo et al. fetal immune atlas Lymphoid Cells GEX data)
- Single-cell VDJ-seq contig data in AIRR format (loadContigs input, e.g. demo_airr)
- Optionally pre-built SCE objects bundled with dandelionR: demo_sce, demo_airr, sce_vdj

## Outputs

- SingleCellExperiment object with combined TCR information in colData ready for setupVdjPseudobulk
- Filtered SCE (e.g. 17,308 cells retained from 65,102 Suo et al. cells) and preprocessing_summary.csv (cells_input, cells_retained)

## How to apply

Load the gene expression matrix (e.g. the Suo et al. fetal-immune atlas Lymphoid Cells GEX data from developmental.cellatlas.io) as a SingleCellExperiment object. Load VDJ contig data with scRepertoire's loadContigs (format = 'AIRR'), combine TCR chains with combineTCR (removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE), and attach receptor information to the SCE colData via combineExpression. The resulting SCE (e.g. demo_sce with sce_vdj-style VDJ metadata) is then passed to dandelionR::setupVdjPseudobulk with already.productive = FALSE and allowed_chain_status = c('Single pair', 'Extra pair', 'Extra pair-exception', 'Orphan VDJ', 'Orphan VDJ-exception') to filter non-productive and ambiguous-chain cells; the vignette demonstrates the same pattern on bundled demo_sce, demo_airr, and sce_vdj objects. Rationale: dandelionR operates directly on SCE objects and interfaces with scRepertoire, so correct SCE construction and colData population is the precondition for downstream vdjPseudobulk, miloR neighbourhood construction, and diffusion-map trajectory steps.

## Related tools

- **dandelionR** (Consumes and manipulates SingleCellExperiment objects; setupVdjPseudobulk filters cells with non-productive immune receptors and ambiguous VDJ chain status on the SCE) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (Provides loadContigs, combineTCR, and combineExpression to merge VDJ contig data into the SCE colData) — https://github.com/ncborcherding/scRepertoire
- **R** (Language/environment in which SingleCellExperiment objects are constructed and manipulated)

## Examples

```
sce <- combineExpression(combineTCR(loadContigs(demo_airr, format = "AIRR"), removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE), demo_sce); sce <- setupVdjPseudobulk(sce, mode_option = "abT", already.productive = FALSE)
```

## Evaluation signals

- Starting cell count is 65,102 for the Suo et al. data; after setupVdjPseudobulk filtering the retained count must equal 17,308 cells with complete TCR information
- colData of the SCE contains combined TCR assignments after combineTCR/combineExpression (removeNA = TRUE, filterMulti = TRUE)
- Running the dandelionR vignette on bundled demo_sce/demo_airr yields an output SCE of 17,281 cells (vs. 17,234 for original Dandelion, 17,208 shared)
- The summary table written to preprocessing_summary.csv contains cells_input and cells_retained columns matching the reported counts

## Limitations

- Downstream destiny-based diffusion map inference only suits bifurcation or linear topologies, constraining what the manipulated SCE can support
- dandelionR's SCE-based workflow has not yet been applied to B cell development data, which involves cyclic receptor-editing trajectories that absorbing Markov chains cannot represent
- SCE objects must contain appropriate reduced dims (e.g. X_scvi) for miloR neighbourhood construction steps (buildGraph k = 30, d = 20)

## Evidence

- [methods] SCE merge with scRepertoire: "sce <- combineExpression(combined.TCR, demo_sce)"
- [methods] loadContigs AIRR format: "contig.list <- loadContigs(input = demo_airr, format = "AIRR")"
- [methods] setupVdjPseudobulk filtering rationale: "This step includes filtering cells with non-productive immune receptors and ambiguous VDJ chain status"
- [results] Suo et al. filtering result: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [intro] dandelionR interfaces with scRepertoire: "dandelionR enables the construction of the VDJ feature space to perform trajectory analysis using diffusion maps and absorbing Markov chains, with seamless interaction with scRepertoire."
