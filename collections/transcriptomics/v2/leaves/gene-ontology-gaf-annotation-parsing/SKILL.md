---
name: gene-ontology-gaf-annotation-parsing
description: Use when DE summary statistics must be paired with pre-defined gene set
  annotations (e.g. for iDEA GSE input) and no annotation matrix matching the required
  gene identifier type exists yet.
when_to_use_negative:
- The iDEA-provided humanGeneSets or mouseGeneSets .RData objects already match the
  identifier type of your summary statistics — no re-parsing needed.
- Your analysis requires tissue/cell-type-specific pathway definitions rather than
  curated public pathway databases.
- Your organism is neither human (GRCh37) nor mouse, as only these annotations are
  covered in the iDEA compilation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: iDEA
  role: Target method whose GSE analysis consumes the parsed binary gene-by-gene-set
    annotation matrices as required inputs
  repo: https://github.com/xzhoulab/iDEA
- name: biomaRt
  role: Identifier conversion to align gene-set matrix row names with the gene naming
    convention of the scRNA-seq summary statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:86aa50135abb22b5e14cdf179a51eefbc8e6b191ed7f6fa6debb10dfd0107582
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - biomaRt
  - iDEA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-ontology-gaf-annotation-parsing

## Summary

Parse public gene set/pathway databases — seven human databases (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome) annotated on GRCh37 from MSigDB and mouse GO annotations in GAF 2.0 format from MGI — into binary gene-by-gene-set annotation matrices suitable for iDEA's GSE analysis.

## When to use

Use when DE summary statistics must be paired with pre-defined gene set annotations (e.g. for iDEA GSE input) and no annotation matrix matching the required gene identifier type exists yet.

## When NOT to use

- The iDEA-provided humanGeneSets or mouseGeneSets .RData objects already match the identifier type of your summary statistics — no re-parsing needed.
- Your analysis requires tissue/cell-type-specific pathway definitions rather than curated public pathway databases.
- Your organism is neither human (GRCh37) nor mouse, as only these annotations are covered in the iDEA compilation.

## Inputs

- Seven human gene set/pathway databases (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome) annotated on GRCh37 from MSigDB
- Mouse gene ontology (GO) annotations in GAF 2.0 format from MGI

## Outputs

- humanGeneSets.RData — binary gene-by-gene-set annotation data.frame with gene symbols as row names (12,033 gene sets)
- mouseGeneSets.RData — binary gene-by-gene-set annotation data.frame with MGI IDs as row names
- Loadable R data.frame objects via data(humanGeneSets) / data(mouseGeneSets)

## How to apply

Download the seven human gene set/pathway databases annotated on reference genome GRCh37 from the MSigDB downloads page, and download mouse GO annotations in GAF 2.0 format from the MGI downloads page. Parse each source into a binary (0/1) gene-by-gene-set annotation data.frame with gene identifiers (gene symbols for human, MGI IDs for mouse) as row names and gene set names as column headers. Remove gene sets whose size falls below the small-gene-set threshold applied in the iDEA compilation (yielding 12,033 human gene sets). Use biomaRt to convert row-name identifiers where needed so they match the identifier type (gene symbol or transcript id) of the scRNA-seq summary statistics they will be paired with. Save the human matrix as humanGeneSets.RData and the mouse matrix as mouseGeneSets.RData, loadable via data(humanGeneSets) and data(mouseGeneSets).

## Related tools

- **iDEA** (Target method whose GSE analysis consumes the parsed binary gene-by-gene-set annotation matrices as required inputs) — https://github.com/xzhoulab/iDEA
- **biomaRt** (Identifier conversion to align gene-set matrix row names with the gene naming convention of the scRNA-seq summary statistics)

## Examples

```
devtools::install_github('xzhoulab/iDEA'); library(iDEA); data(humanGeneSets)
```

## Evaluation signals

- Saved .RData files load in R as data.frame objects with gene names as row names and gene set names as column headers.
- The human compilation contains a total of 12,033 gene sets after pruning.
- Row identifiers match exactly the identifier type (gene symbol or transcript id) of the summary statistics they will be paired with.
- Annotation matrices are strictly binary (0/1) values indicating gene membership per gene set.
- data(humanGeneSets) and data(mouseGeneSets) load the matrices successfully in R.

## Limitations

- Only reference genome GRCh37 (human) and MGI GAF 2.0 (mouse) annotations are covered; other genomes/builds require equivalent sources.
- Gene sets whose size falls below the small-gene-set threshold are removed, so very small gene sets are unavailable.
- iDEA's GSE tests do not explicitly account for gene set overlap that may cause non-independence among gene sets (addressed instead via permuted null / FDR estimation).

## Evidence

- [methods] compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases (http://software.broadinstitute.org/gsea/downloads.jsp): "compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases (http://software.broadinstitute.org/gsea/downloads.jsp)"
- [methods] These databases include BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome.: "These databases include BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome."
- [abstract] iDEA uses only DE summary statistics as input, enabling effective data modeling through complementing and pairing with various existing DE methods: "iDEA uses only DE summary statistics as input, enabling effective data modeling through complementing and pairing with various existing DE methods"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
