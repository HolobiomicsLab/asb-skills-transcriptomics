---
name: cross-dataset-intensity-normalization
description: Use when combining probe-level microarray intensity data from multiple
  GEO series (e.g. 3T3-L1 differentiation studies) where per-study platforms and processing
  differ and cross-study comparability is required.
when_to_use_negative:
- Input is a single-platform, single-study dataset where cross-study normalization
  would add no benefit and could distort within-study biology.
- Input is RNA-seq or ChIP-seq count data (e.g. curatedAdipoRNA/curatedAdipoChIP pipelines)
  rather than microarray probe intensities — this skill addresses array intensity
  harmonization.
- Datasets lack complete probe annotations or probe intensities; these must be excluded
  before normalization rather than imputed into it.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: log transformation and cross-study normalization of gene intensities after
    removing missing and low-intensity genes
- name: sva
  role: removal of known batch effects (data series and platform) after normalization
- name: collapseRows
  role: collapsing probe intensities to gene symbols using probe-to-gene-symbol annotation
- name: GEOquery
  role: retrieval of probe intensities, probe annotations and metadata from GEO for
    each series
- name: SummarizedExperiment
  role: packaging each processed, normalized dataset as an R/Bioconductor SummarizedExperiment
    object
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:c3fa679c32d7e692aafa3da5e6cd44ddde2e3be7064fa1502730f683315011c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - ExperimentHub
  - GEOquery
  - SummarizedExperiment
  - collapseRows
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-dataset-intensity-normalization

## Summary

Harmonize gene-level intensities from multiple public microarray studies so they are comparable across platforms and series: collapse probes to gene symbols, remove missing/low-intensity genes, log transform, normalize across studies with limma, and remove known batch effects with sva.

## When to use

Use when combining probe-level microarray intensity data from multiple GEO series (e.g. 3T3-L1 differentiation studies) where per-study platforms and processing differ and cross-study comparability is required.

## When NOT to use

- Input is a single-platform, single-study dataset where cross-study normalization would add no benefit and could distort within-study biology.
- Input is RNA-seq or ChIP-seq count data (e.g. curatedAdipoRNA/curatedAdipoChIP pipelines) rather than microarray probe intensities — this skill addresses array intensity harmonization.
- Datasets lack complete probe annotations or probe intensities; these must be excluded before normalization rather than imputed into it.

## Inputs

- Per-GEO-series probe intensity matrices (raw/processed microarray intensities)
- Probe-to-gene-symbol annotation tables for each platform
- Study/series/sample metadata (homogenized, with samples classified as none, genetically modified, or pharmacologically modified)

## Outputs

- Gene-symbol-level log-transformed, cross-study normalized intensity matrices
- Batch-effect-corrected expression matrices (series and platform effects removed via sva)
- Per-dataset processed SummarizedExperiment objects plus aggregated genetic and pharmacological perturbation sets

## How to apply

First collapse probe intensities to gene symbols using each platform's probe-to-gene-symbol annotation with collapseRows, dropping datasets whose probe annotations or intensities are incomplete. Then remove missing and low-intensity genes, log transform the gene intensities, and normalize across studies using limma so values share a common scale despite different original platforms. Finally remove known batch effects — the data series and the array platform — using sva. The rationale is that surveyed studies (43 studies, 47 datasets examined, 41 retained) were processed on different platforms, and series/platform are the dominant technical covariates; normalization must precede packaging so each processed dataset is comparable within the aggregated perturbation sets.

## Related tools

- **limma** (log transformation and cross-study normalization of gene intensities after removing missing and low-intensity genes)
- **sva** (removal of known batch effects (data series and platform) after normalization)
- **collapseRows** (collapsing probe intensities to gene symbols using probe-to-gene-symbol annotation)
- **GEOquery** (retrieval of probe intensities, probe annotations and metadata from GEO for each series)
- **SummarizedExperiment** (packaging each processed, normalized dataset as an R/Bioconductor SummarizedExperiment object)

## Examples

```
Rscript -e 'library(GEOquery); library(limma); library(sva); # collapseRows to gene symbols, drop missing/low-intensity genes, log transform, normalize across studies, remove series/platform batch effects'
```

## Evaluation signals

- Dataset retention count matches the article's curation outcome: 47 datasets surveyed, 41 retained after excluding those with missing probe intensities or probe annotations.
- Output intensities are at gene-symbol level (post-collapseRows), with missing and low-intensity genes removed before log transformation.
- Normalized values are on a common scale across studies (limma) and known batch covariates — data series and platform — are no longer detectable in the processed data (sva).
- Each processed dataset is a valid SummarizedExperiment object and the aggregate perturbation sets (genetic and pharmacological) assemble without sample metadata conflicts.
- Pipeline runs reproducibly in the provided Docker image (bcmslab/adiporeg_array) and reproduces the documented biological observations on a data subset.

## Limitations

- Only known batch effects (data series and platform) are removed; unknown or unmodeled confounders may remain.
- Datasets with missing probe intensities or missing probe annotations cannot be normalized and must be excluded (three datasets were dropped in this curation).
- Collapsing to gene symbols discards probe-level resolution and depends on the quality of each platform's probe-to-gene annotation.

## Evidence

- [intro] Collapse probe intensities to gene symbols: "collapsed using probe to gene symbol annotation using `collapsRows`"
- [intro] Normalize across studies with limma after filtering: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] Remove known batch effects with sva: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] Exclusion criteria for datasets: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [readme] Packaging of processed datasets: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
