---
name: binary-gene-set-matrix-construction
description: Use when iDEA-based GSE analysis requires pre-defined gene set annotations
  whose gene identifiers must match the identifier type (gene symbols or MGI IDs)
  of DE summary statistics.
when_to_use_negative:
- Gene set annotations already exist as an iDEA-compatible binary matrix (e.g. the
  distributed humanGeneSets.RData / mouseGeneSets.RData files) — load them instead
  of recompiling
- Summary statistics use identifiers other than gene symbols or MGI IDs and identifier
  conversion via biomaRt is not possible
- The analysis goal is differential expression itself rather than gene set enrichment
  — gene set annotations are not required as input
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: R package whose GSE analysis consumes the compiled binary gene set annotation
    matrices as required input alongside DE summary statistics
  repo: https://github.com/xzhoulab/iDEA
- name: biomaRt
  role: Used for gene identifier conversion to align row names of the gene set matrices
    with the summary statistics' gene naming convention
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:6af363d02ddef10641d44600ca50b6dd3bf3539a3f61bf4af6521728c1cf5d21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - biomaRt
  - iDEA
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# binary-gene-set-matrix-construction

## Summary

Construct binary (0/1) gene-by-gene-set annotation matrices for iDEA's GSE analysis by compiling and pruning human gene sets from seven MSigDB databases (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome on GRCh37) and mouse GO annotations from MGI GAF 2.0, yielding 12,033 usable human gene sets.

## When to use

Use when iDEA-based GSE analysis requires pre-defined gene set annotations whose gene identifiers must match the identifier type (gene symbols or MGI IDs) of DE summary statistics.

## When NOT to use

- Gene set annotations already exist as an iDEA-compatible binary matrix (e.g. the distributed humanGeneSets.RData / mouseGeneSets.RData files) — load them instead of recompiling
- Summary statistics use identifiers other than gene symbols or MGI IDs and identifier conversion via biomaRt is not possible
- The analysis goal is differential expression itself rather than gene set enrichment — gene set annotations are not required as input

## Inputs

- Seven human gene set/pathway databases annotated on GRCh37 from MSigDB (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome)
- Mouse gene ontology annotations in GAF 2.0 format from MGI
- scRNA-seq DE summary statistics whose gene identifier type must be matched (gene symbols or MGI IDs)

## Outputs

- humanGeneSets.RData — binary (0/1) gene-by-gene-set data.frame loadable via data(humanGeneSets)
- mouseGeneSets.RData — binary (0/1) gene-by-gene-set data.frame loadable via data(mouseGeneSets)

## How to apply

Download the seven human gene set/pathway databases annotated on GRCh37 from the MSigDB downloads page and the mouse GO annotations in GAF 2.0 format from MGI. Parse each source into a binary (0/1) data.frame with gene identifiers as row names (symbols for human, MGI IDs for mouse) and gene set names as column headers. Remove gene sets whose set size falls below the small-gene-set threshold applied in the iDEA compilation; use the min_percent_annot coverage-rate parameter (default 0.0025, gene set size divided by number of tested genes) when filtering at analysis time. Align row-name identifier types with the gene naming convention of the scRNA-seq summary statistics using biomaRt for conversion where needed. Save the results as humanGeneSets.RData and mouseGeneSets.RData, loadable via data(humanGeneSets) and data(mouseGeneSets) in R.

## Related tools

- **iDEA** (R package whose GSE analysis consumes the compiled binary gene set annotation matrices as required input alongside DE summary statistics) — https://github.com/xzhoulab/iDEA
- **biomaRt** (Used for gene identifier conversion to align row names of the gene set matrices with the summary statistics' gene naming convention)

## Examples

```
devtools::install_github('xzhoulab/iDEA'); library(iDEA); data(humanGeneSets)
```

## Evaluation signals

- Saved .RData files load in R as data.frame objects with gene names as row names
- Row identifiers of the gene set matrices match exactly the identifier type (gene symbol or MGI ID/transcript id) of the summary statistics they will be paired with
- Human matrix contains the pruned gene sets compiled from the seven MSigDB databases; article reports 12,033 gene sets after pruning
- Matrix entries are strictly binary (0/1) indicating gene membership in each gene set
- data(humanGeneSets) and data(mouseGeneSets) load successfully in R after installing iDEA via devtools::install_github('xzhoulab/iDEA')

## Limitations

- iDEA's compilation provides only human and mouse gene set annotations; other organisms are not covered
- Gene sets whose size falls below the compilation threshold are removed, so very small gene sets are unavailable
- Identifier conversion via biomaRt can be a point of failure if the summary statistics use transcript ids or non-standard symbol versions that do not align exactly

## Evidence

- [methods] Compilation of seven human MSigDB databases: "compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases (http://software.broadinstitute.org/gsea/downloads.jsp)"
- [methods] Source databases enumerated: "These databases include BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome."
- [methods] Coverage-rate filter parameter for gene set size: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [methods] Pre-built gene set data files distributed with iDEA: "downloaded [here](https://github.com/xzhoulab/iDEA/tree/master/data/humanGeneSets.RData) for human gene sets"
- [readme] iDEA installation via GitHub enabling data() loading of gene sets: "devtools::install_github('xzhoulab/iDEA')"
