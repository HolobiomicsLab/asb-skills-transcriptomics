---
name: gene-set-size-filtering
description: Use when preparing gene-by-gene-set annotation matrices (e.g. binary
  0/1 data.frames from MSigDB or MGI GO GAF 2.0 files) as inputs for iDEA GSE analysis,
  and sparse sets below the size threshold must be removed first.
when_to_use_negative:
- Gene set annotations are already provided as iDEA's packaged humanGeneSets/mouseGeneSets
  objects — do not re-download and re-prune
- The downstream GSE method expects full unfiltered pathway databases rather than
  iDEA's curated 12,033 gene sets
- Summary statistics use identifier types that cannot be mapped to gene symbols or
  MGI IDs, since pruning plus alignment requires matchable row names
tools:
- name: iDEA
  role: Target framework whose compiled annotation files define the small-gene-set
    threshold applied when pruning the source databases
  repo: https://github.com/xzhoulab/iDEA
- name: biomaRt
  role: Used for identifier conversion so pruned gene set matrix row names match the
    gene symbols or transcript IDs in the scRNA-seq summary statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:f77f6389b9ddc425f501cf453eb8df60eb70bd8da44515c3d2b3452b7021b49e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - biomaRt
  - iDEA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# gene-set-size-filtering

## Summary

Prunes compiled gene-set/pathway annotation matrices (human MSigDB-derived sets and mouse GO annotations) by a minimum gene-set size threshold so that only gene sets large enough to be usable in iDEA's GSE analysis are retained. This prevents sparse, underpowered gene sets from entering downstream enrichment testing.

## When to use

Use when preparing gene-by-gene-set annotation matrices (e.g. binary 0/1 data.frames from MSigDB or MGI GO GAF 2.0 files) as inputs for iDEA GSE analysis, and sparse sets below the size threshold must be removed first.

## When NOT to use

- Gene set annotations are already provided as iDEA's packaged humanGeneSets/mouseGeneSets objects — do not re-download and re-prune
- The downstream GSE method expects full unfiltered pathway databases rather than iDEA's curated 12,033 gene sets
- Summary statistics use identifier types that cannot be mapped to gene symbols or MGI IDs, since pruning plus alignment requires matchable row names

## Inputs

- MSigDB human gene set downloads for BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome annotated on GRCh37
- Mouse Gene Ontology annotations in GAF 2.0 format from MGI
- Gene-set size threshold (e.g. iDEA's min_percent_annot coverage-rate cutoff, default 0.0025)

## Outputs

- humanGeneSets.RData — pruned binary gene-by-gene-set data.frame with gene symbols as row names, loadable via data(humanGeneSets)
- mouseGeneSets.RData — pruned binary gene-by-gene-set data.frame with MGI IDs as row names, loadable via data(mouseGeneSets)

## How to apply

Download the seven human gene set/pathway databases (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome) annotated on GRCh37 from the MSigDB downloads page, and mouse GO annotations in GAF 2.0 format from MGI. Parse each source into a binary (0/1) gene-by-gene-set data.frame with gene identifiers (symbols for human, MGI IDs for mouse) as row names and gene set names as columns. Remove gene sets whose set size (number of annotated genes) falls below the small-gene-set threshold applied in the iDEA compilation — conceptually matching iDEA's min_percent_annot coverage-rate criterion (annotated genes divided by tested genes, default 0.0025) — so only usable sets remain. Align row-name identifier types with the gene naming convention of the paired scRNA-seq summary statistics (using biomaRt conversion where needed), then save as humanGeneSets.RData / mouseGeneSets.RData loadable via data().

## Related tools

- **iDEA** (Target framework whose compiled annotation files define the small-gene-set threshold applied when pruning the source databases) — https://github.com/xzhoulab/iDEA
- **biomaRt** (Used for identifier conversion so pruned gene set matrix row names match the gene symbols or transcript IDs in the scRNA-seq summary statistics)

## Examples

```
library(iDEA); data(humanGeneSets); data(mouseGeneSets)
```

## Evaluation signals

- Saved .RData files load in R as data.frame objects with gene names as row names and gene set names as column headers
- Row identifiers of the matrices match exactly the identifier type (gene symbol or transcript id) of the summary statistics they will be paired with
- No retained gene set has a set size below the small-gene-set threshold applied in the iDEA compilation
- The human matrix reduces the seven GRCh37-annotated MSigDB source databases to iDEA's total of 12,033 usable gene sets
- Both matrices load reproducibly via data(humanGeneSets) and data(mouseGeneSets) after devtools::install_github('xzhoulab/iDEA')

## Limitations

- iDEA's own enrichment testing does not explicitly account for gene set overlap that may cause non-independence among gene sets, so pruning by size alone does not remove redundancy between sets
- Gene-set identifiers and annotation builds (GRCh37 for human; GAF 2.0 mouse GO) are version-specific, and identifier conversion via biomaRt is needed when summary statistics use different naming conventions
- The specific small-gene-set cutoff used in the iDEA compilation is applied as a fixed compilation choice rather than being re-tuned per analysis

## Evidence

- [methods] MSigDB source databases: "compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases"
- [methods] source database list: "These databases include BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome."
- [methods] coverage-rate threshold concept: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [other] pruning step from task workflow: [evidence span withheld — anchor status: not_found]
- [readme] installation/loadable artifacts: "devtools::install_github('xzhoulab/iDEA')"
