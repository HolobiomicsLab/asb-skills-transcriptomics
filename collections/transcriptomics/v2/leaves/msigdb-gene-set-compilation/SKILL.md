---
name: msigdb-gene-set-compilation
description: Use when iDEA GSE analysis is planned and a gene set annotation matrix
  is needed that matches the gene identifier type (symbols or MGI IDs) of the DE summary
  statistics.
when_to_use_negative:
- When a custom gene set collection already exists in the correct identifier space
  — compiling from MSigDB would duplicate or override it
- When the summary statistics use identifier types (e.g. Ensembl IDs) that cannot
  be matched to the MSigDB/MGI row names even after biomaRt conversion
- When performing over-representation analysis tools like DAVID or Fisher's exact
  test that take gene lists, not full annotation matrices
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: iDEA
  role: Target R package whose GSE module consumes the compiled humanGeneSets/mouseGeneSets
    annotation matrices as input
  repo: https://github.com/xzhoulab/iDEA
- name: biomaRt
  role: Identifier conversion to align gene set matrix row names with summary-statistic
    gene naming
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:0ec4780d5d2bda72f623358845c45e12d039e1169c026f3288ff55bc035f8653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - biomaRt
  - iDEA
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# msigdb-gene-set-compilation

## Summary

Compile iDEA's pre-defined gene set annotations by merging seven human pathway databases from MSigDB (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome) and mouse GO from MGI into binary gene-by-gene-set matrices, pruned to usable set sizes and saved as loadable .RData objects.

## When to use

Use when iDEA GSE analysis is planned and a gene set annotation matrix is needed that matches the gene identifier type (symbols or MGI IDs) of the DE summary statistics.

## When NOT to use

- When a custom gene set collection already exists in the correct identifier space — compiling from MSigDB would duplicate or override it
- When the summary statistics use identifier types (e.g. Ensembl IDs) that cannot be matched to the MSigDB/MGI row names even after biomaRt conversion
- When performing over-representation analysis tools like DAVID or Fisher's exact test that take gene lists, not full annotation matrices

## Inputs

- MSigDB human gene set/pathway files on GRCh37 (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome)
- Mouse GO annotations in GAF 2.0 format from MGI
- Target gene identifier type of the DE summary statistics (gene symbols or MGI IDs)

## Outputs

- humanGeneSets.RData: binary gene-by-gene-set data.frame (gene symbols as row names, 12,033 gene sets)
- mouseGeneSets.RData: binary gene-by-gene-set data.frame (MGI IDs as row names)

## How to apply

Download the seven human gene set/pathway databases annotated on GRCh37 from the MSigDB downloads page (http://software.broadinstitute.org/gsea/downloads.jsp) and the mouse GO annotations in GAF 2.0 format from MGI. Parse each source into a binary (0/1) gene-by-gene-set data.frame with gene symbols (human) or MGI IDs (mouse) as row names and gene set names as columns. Remove gene sets whose size falls below the small-gene-set threshold used in the iDEA compilation, yielding 12,033 human gene sets in total. Align row-name identifier types with the summary statistics (using biomaRt conversion where needed) and save as humanGeneSets.RData / mouseGeneSets.RData, loadable via data(humanGeneSets).

## Related tools

- **iDEA** (Target R package whose GSE module consumes the compiled humanGeneSets/mouseGeneSets annotation matrices as input) — https://github.com/xzhoulab/iDEA
- **biomaRt** (Identifier conversion to align gene set matrix row names with summary-statistic gene naming)

## Examples

```
library(iDEA); data(humanGeneSets)
```

## Evaluation signals

- The saved .RData files load in R as data.frame objects via data(humanGeneSets) / data(mouseGeneSets)
- Row names are gene symbols (human) or MGI IDs (mouse), and match exactly the identifier type of the paired summary statistics
- Entries are binary 0/1 annotations with gene set names as column headers
- The pruned human matrix contains 12,033 gene sets aggregated from the seven source databases
- No gene sets remain below the small-gene-set size threshold applied during iDEA compilation

## Limitations

- Compilation is tied to reference genome GRCh37 for human annotations; other genome builds require re-download and re-alignment
- Gene sets whose size falls below the small-gene-set threshold are discarded and unavailable for GSE testing
- Gene sets are not independent (they overlap), so GSE tests on the compiled annotation require permutation-based FDR (iDEA.fit.null / iDEA.FDR) to account for non-independence

## Evidence

- [methods] compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases: "compiled seven exsiting gene set/pathway databases annotated on the reference genome GRCh37 from MSigDB databases (http://software.broadinstitute.org/gsea/downloads.jsp)"
- [methods] seven source databases listed: "These databases include BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID and Reactome."
- [methods] loading the compiled human annotation: [evidence span withheld — anchor status: not_found]
- [readme] installing the consuming package: "devtools::install_github('xzhoulab/iDEA')"
