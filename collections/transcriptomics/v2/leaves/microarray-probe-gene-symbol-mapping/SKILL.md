---
name: microarray-probe-gene-symbol-mapping
description: Use when a downloaded GEO microarray series provides probe-level intensities
  with a probe-to-gene annotation table, and a gene-by-sample expression matrix (one
  row per unique gene symbol) is required for downstream analysis.
when_to_use_negative:
- The input is already a gene-level expression matrix (one row per gene symbol) —
  collapsing again is unnecessary and could discard data.
- The GEO dataset has missing probe intensities or missing probe annotations — such
  studies were excluded in this curation because the probe-to-gene mapping cannot
  be performed reliably.
- The data are count-based RNA-seq reads rather than probe-level microarray intensities
  — use alignment/quantification (e.g. the curatedAdipoRNA RangedSummarizedExperiment
  route) instead of probe collapsing.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: GEOquery
  role: Downloads probe intensities, probe annotations and metadata for each GEO series
    before mapping probes to gene symbols
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: collapseRows
  role: Collapses the probe-level intensity matrix to one row per gene symbol using
    the MaxMean rule
- name: curatedAdipoArray
  role: R/Bioconductor package containing the scripts (inst/scripts) that document
    and reproduce the probe-to-gene collapsing pipeline
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: Docker
  role: Provides the pre-processing environment (bcmslab/adiporeg_array image) in
    which the collapsing pipeline is reproduced
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:379ee6976a974b1ce48d7ccb65a4d6012ad8edd4e8c890d87c78648ee8d51b53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker
  - ExperimentHub
  - GEOquery
  - R/Bioconductor
  - collapseRows
  - curatedAdipoArray
  - sva
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# microarray-probe-gene-symbol-mapping

## Summary

Collapses probe-level microarray intensity matrices to gene-level expression by matching probe identifiers to probe-to-gene-symbol annotations and merging multi-probe genes with the MaxMean rule of WGCNA's collapseRows, as used to curate 41 GEO 3T3-L1 microarray datasets for the curatedAdipoArray resource.

## When to use

Use when a downloaded GEO microarray series provides probe-level intensities with a probe-to-gene annotation table, and a gene-by-sample expression matrix (one row per unique gene symbol) is required for downstream analysis.

## When NOT to use

- The input is already a gene-level expression matrix (one row per gene symbol) — collapsing again is unnecessary and could discard data.
- The GEO dataset has missing probe intensities or missing probe annotations — such studies were excluded in this curation because the probe-to-gene mapping cannot be performed reliably.
- The data are count-based RNA-seq reads rather than probe-level microarray intensities — use alignment/quantification (e.g. the curatedAdipoRNA RangedSummarizedExperiment route) instead of probe collapsing.

## Inputs

- GEO series matrix of probe-level microarray intensities (obtained via GEOquery)
- Probe-to-gene-symbol annotation table for the array platform
- Sample metadata (sample accessions) from the GEO series

## Outputs

- Gene-by-sample expression matrix with unique gene symbols as row identifiers and sample accessions as columns (e.g. gene_level_expression.csv)
- One row per unique gene symbol after MaxMean probe collapsing

## How to apply

Retrieve the series matrix (probe intensities, probe annotation, sample metadata) from GEO with GEOquery in R/Bioconductor, and match each probe identifier to its mouse gene symbol from the platform's probe-to-gene annotation. Run WGCNA's collapseRows with method='MaxMean', which selects for each gene the probe (or probe set) with the highest mean expression across samples so that multiple probes mapping to one gene collapse to a single row. Write the collapsed gene-by-sample matrix with gene symbols as row identifiers and sample accessions as columns (e.g. gene_level_expression.csv). Before collapsing, exclude datasets with missing probe intensities or missing probe annotations, since the collapse is only meaningful when both are complete; afterwards, remove missing and low-intensity genes, then log transform and normalize across studies with limma.

## Related tools

- **GEOquery** (Downloads probe intensities, probe annotations and metadata for each GEO series before mapping probes to gene symbols) — https://github.com/MahShaaban/curatedAdipoArray
- **collapseRows** (Collapses the probe-level intensity matrix to one row per gene symbol using the MaxMean rule)
- **curatedAdipoArray** (R/Bioconductor package containing the scripts (inst/scripts) that document and reproduce the probe-to-gene collapsing pipeline) — https://github.com/MahShaaban/curatedAdipoArray
- **Docker** (Provides the pre-processing environment (bcmslab/adiporeg_array image) in which the collapsing pipeline is reproduced)

## Examples

```
collapsed <- WGCNA::collapseRows(datExpr = probe_intensities, rowGroup = probe_annotation$gene_symbol, rowID = probe_annotation$probe_id, method = "MaxMean")
```

## Evaluation signals

- Every probe in the input matrix is assigned a mouse gene symbol from the annotation table before collapsing — no unmapped probes are silently dropped.
- The collapsed matrix has exactly one row per unique gene symbol, with no duplicated gene identifiers.
- For genes with multiple probes, the retained value corresponds to the probe (or probe set) with the highest mean expression across samples, consistent with the MaxMean rule.
- Column identifiers of the collapsed matrix match the sample accessions in the GEO series metadata.
- Downstream filtering of missing and low-intensity genes followed by log transformation and limma normalization succeeds on the collapsed matrix without identifier errors.

## Limitations

- Studies/datasets with missing probe intensities or missing probe annotations must be excluded — two datasets were excluded for missing probe annotations, so the collapse cannot rescue incomplete annotation.
- The MaxMean rule discards the intensity information of all non-maximal probes for a multi-probe gene, which is a lossy summarization choice.
- Gene symbols must be matched to the correct platform's probe-to-gene annotation; the pipeline is specific to mouse 3T3-L1 microarray datasets surveyed here.

## Evidence

- [abstract] Probe intensities were mapped and collapsed to genes using a reproducible pipeline.: "Probe intensities were mapped and collapsed to genes using a reproducible pipeline."
- [intro] Collapse probe intensities to gene symbols via collapseRows: "collapsed using probe to gene symbol annotation using `collapsRows`"
- [intro] GEO retrieval of probe intensities, annotations and metadata: "The probe intensities, probe annotation and metadata of each study were obtained from the gene expression omnibus (GEO)"
- [methods] MaxMean rule from task card workflow: [evidence span withheld — anchor status: not_found]
- [intro] Exclusion of datasets with incomplete annotations: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [readme] Package scripts reproduce the data: "1. Scripts for documenting and reproducing the dataset in `inst/scripts`."
