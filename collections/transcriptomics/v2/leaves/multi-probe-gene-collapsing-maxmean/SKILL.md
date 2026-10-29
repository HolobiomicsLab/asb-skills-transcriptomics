---
name: multi-probe-gene-collapsing-maxmean
description: Use when you have a probe-level microarray intensity matrix (e.g. a GEO
  series matrix) whose annotation table maps multiple probes to the same gene symbol,
  and you need one row per gene for downstream analysis.
when_to_use_negative:
- The data are already gene-level (e.g. RNA-seq counts like the curatedAdipoRNA adipo_counts
  matrix) — no probe collapsing is needed
- The dataset lacks a probe-to-gene annotation table — in the source study, datasets
  missing probe annotations were excluded rather than collapsed
tools:
- name: collapseRows
  role: Collapses the probe-level intensity matrix to one row per unique gene symbol
    using the MaxMean rule (cited in the article as `collapsRows`)
- name: GEOquery
  role: Retrieves the probe intensities, probe annotation and sample metadata of each
    study from the Gene Expression Omnibus before collapsing
- name: curatedAdipoArray
  role: Bioconductor package whose inst/scripts document and reproduce the probe-to-gene
    collapsing pipeline for the 3T3-L1 microarray data
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: R/Bioconductor
  role: Execution environment for the GEOquery retrieval and collapseRows collapsing
    steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:b300e47eeea60944d63019f14875c5e5aa0a398dfcc160631fc39876f6e02000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - GEOquery
  - R/Bioconductor
  - collapseRows
  - curatedAdipoArray
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multi-probe-gene-collapsing-maxmean

## Summary

Collapses a probe-level microarray intensity matrix to a gene-level matrix by mapping probe IDs to mouse gene symbols and, for genes with multiple probes, applying the 'MaxMean' rule of the collapseRows function to retain the probe with the highest mean expression across samples.

## When to use

Use when you have a probe-level microarray intensity matrix (e.g. a GEO series matrix) whose annotation table maps multiple probes to the same gene symbol, and you need one row per gene for downstream analysis.

## When NOT to use

- The data are already gene-level (e.g. RNA-seq counts like the curatedAdipoRNA adipo_counts matrix) — no probe collapsing is needed
- The dataset lacks a probe-to-gene annotation table — in the source study, datasets missing probe annotations were excluded rather than collapsed

## Inputs

- Probe-level intensity matrix from a GEO series matrix file (probe IDs x sample accessions)
- Probe-to-gene-symbol annotation table for the microarray platform
- Sample metadata/accessions retrieved via GEOquery

## Outputs

- Gene-level expression matrix with one row per unique gene symbol and sample accessions as columns
- gene_level_expression.csv with gene symbols as row identifiers

## How to apply

Retrieve the GEO series matrix (probe intensities, probe annotation and sample metadata) with GEOquery, then match each probe identifier to its probe-to-gene-symbol annotation so every probe is assigned a mouse gene symbol. Run collapseRows with method='MaxMean' (the article cites it as `collapsRows`) on the probe intensity matrix with the gene-symbol grouping, so each unique gene symbol is represented by the probe set with the highest mean expression across samples. Write the collapsed gene-by-sample matrix, with gene symbols as row identifiers and sample accessions as columns, to gene_level_expression.csv. This step precedes removal of missing/low-intensity genes, log transformation, and cross-study normalization with limma in the curatedAdipoArray pipeline.

## Related tools

- **collapseRows** (Collapses the probe-level intensity matrix to one row per unique gene symbol using the MaxMean rule (cited in the article as `collapsRows`))
- **GEOquery** (Retrieves the probe intensities, probe annotation and sample metadata of each study from the Gene Expression Omnibus before collapsing)
- **curatedAdipoArray** (Bioconductor package whose inst/scripts document and reproduce the probe-to-gene collapsing pipeline for the 3T3-L1 microarray data) — https://github.com/MahShaaban/curatedAdipoArray
- **R/Bioconductor** (Execution environment for the GEOquery retrieval and collapseRows collapsing steps)

## Examples

```
collapsed <- collapseRows(probe_matrix, rowGroup = annotation$gene_symbol, rowID = annotation$probe_id, method = "MaxMean")$datET collapsed
```

## Evaluation signals

- The collapsed matrix has exactly one row per unique gene symbol, with no duplicated gene identifiers
- For every multi-probe gene, the retained probe corresponds to the one with the highest mean expression across samples (MaxMean rule)
- Column identifiers of the output match the original sample accessions (no samples lost or reordered during collapsing)
- gene_level_expression.csv contains gene symbols as row identifiers and sample accessions as columns
- Genes whose probes are missing annotation (the reason two studies were excluded) are not silently dropped

## Limitations

- Collapsing requires a complete probe-to-gene-symbol annotation; in the source curation, two of the 47 examined datasets were excluded for missing probe annotations
- One study was excluded for missing probe intensities, so the input matrix must have usable intensity values before collapsing
- The MaxMean rule discards all but the best-mean probe per gene, so probe-level variability is not averaged or summarized across probes

## Evidence

- [intro] Probe-to-gene collapsing is performed via collapseRows: "collapsed using probe to gene symbol annotation using `collapsRows`"
- [abstract] The pipeline maps and collapses probe intensities reproducibly: "Probe intensities were mapped and collapsed to genes using a reproducible pipeline."
- [intro] Input data are obtained from GEO with GEOquery: "obtained from the gene expression omnibus (GEO) using `GEOquery`"
- [readme] The curatedAdipoArray package ships the scripts that reproduce the data processing: "The package documents the data collection and processing. In addition to the documentation, the package contains the scripts that was used to generated the data."
