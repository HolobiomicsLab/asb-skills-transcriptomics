---
name: geo-series-data-retrieval
description: Use when starting from a GEO accession (GSE series) and you need the
  raw probe-level intensity matrix, its probe annotation table, and per-sample metadata
  to feed a curation or normalization pipeline.
when_to_use_negative:
- You only need the already-processed gene-level data, which can be fetched as SummarizedExperiment
  objects via ExperimentHub from curatedAdipoArray
- The input is sequencing-derived gene counts (e.g., the curatedAdipoRNA RNA-Seq dataset)
  rather than microarray probe intensities
- The target GEO study lacks probe annotations or probe intensities — in the curatedAdipoArray
  curation such studies were excluded outright
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEOquery
  role: R/Bioconductor package used to download probe intensities, probe annotation,
    and metadata of each study from GEO
- name: R/Bioconductor
  role: Runtime environment in which the GEO retrieval and downstream SummarizedExperiment
    packaging are executed
- name: curatedAdipoArray
  role: Bioconductor experimental data package that documents the retrieval pipeline
    and distributes the retrieved/processed data via ExperimentHub
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Distribution channel through which the retrieved and processed SummarizedExperiment
    objects are accessed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:adbc50d5a629bb811fc4c295ba743f8745265b52563887205a6ab1d2e4447bb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - GEOquery
  - R/Bioconductor
  - collapseRows
  - curatedAdipoArray
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# geo-series-data-retrieval

## Summary

Retrieves probe intensities, probe-to-gene annotation tables, and sample metadata directly from GEO series matrices (e.g., GSE12929) using GEOquery in R/Bioconductor, as the first step of a microarray curation pipeline before probe-to-gene collapsing.

## When to use

Use when starting from a GEO accession (GSE series) and you need the raw probe-level intensity matrix, its probe annotation table, and per-sample metadata to feed a curation or normalization pipeline.

## When NOT to use

- You only need the already-processed gene-level data, which can be fetched as SummarizedExperiment objects via ExperimentHub from curatedAdipoArray
- The input is sequencing-derived gene counts (e.g., the curatedAdipoRNA RNA-Seq dataset) rather than microarray probe intensities
- The target GEO study lacks probe annotations or probe intensities — in the curatedAdipoArray curation such studies were excluded outright

## Inputs

- GEO series accession (e.g., GSE12929) for an MDI-induced 3T3-L1 microarray study
- GEO series matrix files containing probe intensities, probe annotation, and sample metadata

## Outputs

- Probe-level intensity matrix (probes x samples) keyed by GEO sample accessions
- Probe-to-gene-symbol annotation table
- Sample metadata table for downstream homogenization into unified vocabularies

## How to apply

Pull the full GSE series matrix (probe intensities, probe annotation, sample metadata) from the Gene Expression Omnibus with GEOquery in R/Bioconductor, rather than re-processing author-supplied raw files. Check completeness of the returned objects immediately: probe intensities and probe-to-gene-symbol annotation must both be present, because datasets missing either were excluded in the curatedAdipoArray curation (one study excluded for missing probe intensities, two for missing probe annotations). Keep the GEO sample accessions as the column identifiers of the intensity matrix so that downstream metadata homogenization and the collapseRows MaxMean step can reference samples consistently. The retrieved expression data are subsequently packaged as SummarizedExperiment objects distributed via ExperimentHub in the curatedAdipoArray package.

## Related tools

- **GEOquery** (R/Bioconductor package used to download probe intensities, probe annotation, and metadata of each study from GEO)
- **R/Bioconductor** (Runtime environment in which the GEO retrieval and downstream SummarizedExperiment packaging are executed)
- **curatedAdipoArray** (Bioconductor experimental data package that documents the retrieval pipeline and distributes the retrieved/processed data via ExperimentHub) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Distribution channel through which the retrieved and processed SummarizedExperiment objects are accessed)

## Examples

```
library(GEOquery); gse <- getGEO("GSE12929", GSEMatrix = TRUE)
```

## Evaluation signals

- The retrieved series matrix contains three aligned components: probe intensities, probe annotation, and sample metadata for the same sample set
- Column names of the intensity matrix are GEO sample accessions matching the metadata rows
- Every probe id in the intensity matrix can be matched to a gene symbol in the annotation table; unmatched probes flag a missing-annotation exclusion case
- Metadata fields are sufficient for later homogenization into common vocabularies describing the experimental design (time point, perturbation type)
- The retrieved data can be wrapped into a SummarizedExperiment object consistent with what curatedAdipoArray exposes via ExperimentHub

## Limitations

- Some GEO studies are incomplete: one study was excluded for missing probe intensities and two for missing probe annotations, so retrieval success does not guarantee usability
- Metadata language varies across studies and must be manually curated into unified vocabularies before samples can be combined
- Retrieval yields probe-level data only; collapsing to gene symbols (collapseRows MaxMean) is a separate downstream step

## Evidence

- [intro] The probe intensities, probe annotation and metadata of each study were obtained from the gene expression omnibus (GEO): "The probe intensities, probe annotation and metadata of each study were obtained from the gene expression omnibus (GEO)"
- [intro] obtained from the gene expression omnibus (GEO) using `GEOquery`: "obtained from the gene expression omnibus (GEO) using `GEOquery`"
- [intro] One and two studies were excluded for missing probe intensities and probe annotations respectively.: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [readme] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects
through `ExperimentHub`."
