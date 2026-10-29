---
name: geo-microarray-download-and-metadata-curation
description: Use when a meta-analysis needs to harvest multiple public microarray
  studies from GEO (series matrix or raw probe intensities) and requires unified,
  cross-study sample metadata before normalization or batch correction.
when_to_use_negative:
- Input is a single pre-processed expression matrix with no GEO series/metadata heterogeneity
  to reconcile
- The source data is sequencing-based (RNA-seq/ChIP-seq) rather than microarray —
  the authors distribute those as separate curatedAdipoRNA/curatedAdipoChIP resources
- Datasets lack probe annotations or have incomplete probe intensities — these are
  excluded by design rather than imputed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: GEOquery
  role: R/Bioconductor package used to download probe intensities, probe annotations,
    and metadata for each retained GEO series
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: curatedAdipoArray
  role: R/Bioconductor experimental data package that documents, scripts, and distributes
    the curated datasets and metadata through ExperimentHub
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: Docker
  role: Provides the reproducible pre-processing/processing environment for the curation
    pipeline (bcmslab/adiporeg_array image)
  repo: https://github.com/MahShaaban/curatedAdipoArray
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:84945ee2dbaea17017c76bd22170379f206c484e5141d89f998f94bc53ae521c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker
  - ExperimentHub
  - GEOquery
  - collapseRows
  - curatedAdipoArray
  - sva
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# geo-microarray-download-and-metadata-curation

## Summary

Retrieves probe intensities, probe annotations, and metadata for public GEO microarray series (e.g. 41 curated 3T3-L1 adipocyte datasets) using GEOquery, then homogenizes study/series/sample metadata across studies with common vocabularies so heterogeneous studies can be pooled.

## When to use

Use when a meta-analysis needs to harvest multiple public microarray studies from GEO (series matrix or raw probe intensities) and requires unified, cross-study sample metadata before normalization or batch correction.

## When NOT to use

- Input is a single pre-processed expression matrix with no GEO series/metadata heterogeneity to reconcile
- The source data is sequencing-based (RNA-seq/ChIP-seq) rather than microarray — the authors distribute those as separate curatedAdipoRNA/curatedAdipoChIP resources
- Datasets lack probe annotations or have incomplete probe intensities — these are excluded by design rather than imputed

## Inputs

- GEO series accessions for published microarray studies
- GEO probe intensity matrices per series
- GEO probe-to-gene annotation tables
- GEO sample/study metadata (series and sample descriptions)

## Outputs

- Per-dataset curated metadata with homogenized vocabularies and sample class (none / genetically modified / pharmacologically modified)
- Set of retained GEO series (41 datasets) with complete probe intensities and annotations
- Downloaded probe intensities and probe annotations ready for collapseRows-based gene mapping

## How to apply

Survey the literature for microarray studies of the target model (here MDI-induced 3T3-L1 at different differentiation time points, with or without perturbation), then download probe intensities, probe annotations, and metadata for each GEO series with the GEOquery package. Exclude datasets that have missing probe intensities or missing probe annotations (in this survey 47 datasets were examined and 41 were retained after exclusion). Homogenize study, series, and sample metadata across studies using common vocabularies, classifying each sample as none, genetically modified, or pharmacologically modified, since inconsistent terminology across 43 published studies prevents direct pooling. Keep the retrieval and curation scripted and versioned (scripts in inst/scripts, reproducible Docker environment) so the assembled SummarizedExperiment objects can be regenerated and distributed via ExperimentHub.

## Related tools

- **GEOquery** (R/Bioconductor package used to download probe intensities, probe annotations, and metadata for each retained GEO series) — https://github.com/MahShaaban/curatedAdipoArray
- **curatedAdipoArray** (R/Bioconductor experimental data package that documents, scripts, and distributes the curated datasets and metadata through ExperimentHub) — https://github.com/MahShaaban/curatedAdipoArray
- **Docker** (Provides the reproducible pre-processing/processing environment for the curation pipeline (bcmslab/adiporeg_array image)) — https://github.com/MahShaaban/curatedAdipoArray

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray"); docker pull bcmslab/adiporeg_array:latest
```

## Evaluation signals

- Number of retained datasets matches the reported 41 of 47 surveyed GEO datasets after excluding those with missing probe intensities (1–2 studies) or missing probe annotations (2 studies)
- Every retained sample carries a homogenized class label: none, genetically modified, or pharmacologically modified
- Each dataset's metadata fields (study, series, sample) use common vocabularies consistent across all included studies
- Per-dataset probe intensities and probe-to-gene annotation tables are present and non-empty for all retained series
- Curation scripts in inst/scripts and the Docker environment (bcmslab/adiporeg_array) regenerate identical metadata/counts

## Limitations

- Datasets with missing probe intensities or missing probe annotations must be dropped (1–2 studies were excluded here), reducing coverage
- Metadata homogenization is manual and depends on unified vocabularies across 43 published studies, so classification choices may not transfer verbatim to other models
- Only MDI-induced 3T3-L1 microarray studies sampled at differentiation time points are in scope; the resource does not cover RNA-seq or ChIP-seq adipocyte data

## Evidence

- [intro] GEOquery retrieves intensities and metadata: "The probe intensities, probe annotation and metadata of each study were obtained from the gene expression omnibus (GEO)"
- [intro] Exclusion criteria for incomplete datasets: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [abstract] Metadata homogenization across studies: "The metadata were manually curated using unified language across the studies."
- [abstract] Sample classification vocabulary: "Samples were classified into none, genetically or pharmacologically modified."
- [intro] Survey scope of the resource: "43 published studies were included. 47 datasets from these studies were examined for metadata and annotation completeness."
- [readme] Reproducible scripting and packaging: "1. Scripts for documenting and reproducing the dataset in `inst/scripts`.
2. Access to the clean and the processed data `SummarizedExperiment` objects
through `ExperimentHub`."
