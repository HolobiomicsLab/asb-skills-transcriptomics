---
name: summarizedexperiment-data-handling
description: Use when a processed, gene-level expression matrix must be packaged with
  its homogenized sample metadata as a SummarizedExperiment object, or when retrieving
  the curatedAdipoArray datasets through ExperimentHub.
when_to_use_negative:
- When the input is still raw GEO probe intensities or uncurated metadata — the curation
  pipeline (filtering, collapseRows, limma, sva) must run before packaging
- When the analysis requires plain matrix files or non-Bioconductor formats (e.g.
  CSV/AnnData) rather than SummarizedExperiment containers
- When working with RNA-seq count or ChIP-seq peak data from the sibling packages,
  which use RangedSummarizedExperiment with rowRanges rather than the microarray-oriented
  objects
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SummarizedExperiment
  role: Container class used to package each processed 3T3-L1 microarray dataset individually
    with its expression assay and curated colData
- name: ExperimentHub
  role: Distribution channel providing access to the clean and processed SummarizedExperiment
    objects
- name: curatedAdipoArray
  role: Bioconductor experimental data package that documents, hosts, and exposes
    the packaged SummarizedExperiment objects and their generation scripts
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: BiocManager
  role: Installer for the curatedAdipoArray package from Bioconductor
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:96df2e8247425314c30d2b0b36b8e5233d16396b686609ea32546b971ae6f813
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - Docker
  - ExperimentHub
  - GEOquery
  - R/Bioconductor
  - SummarizedExperiment
  - collapseRows
  - curatedAdipoArray
  - sva
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# summarizedexperiment-data-handling

## Summary

Packages curated, processed gene-level expression data (e.g. the batch-corrected 3T3-L1 adipocyte microarray matrices from curatedAdipoArray) as R/Bioconductor SummarizedExperiment objects and distributes them for downstream reuse via ExperimentHub.

## When to use

Use when a processed, gene-level expression matrix must be packaged with its homogenized sample metadata as a SummarizedExperiment object, or when retrieving the curatedAdipoArray datasets through ExperimentHub.

## When NOT to use

- When the input is still raw GEO probe intensities or uncurated metadata — the curation pipeline (filtering, collapseRows, limma, sva) must run before packaging
- When the analysis requires plain matrix files or non-Bioconductor formats (e.g. CSV/AnnData) rather than SummarizedExperiment containers
- When working with RNA-seq count or ChIP-seq peak data from the sibling packages, which use RangedSummarizedExperiment with rowRanges rather than the microarray-oriented objects

## Inputs

- Processed gene-level expression matrix (limma-normalized, sva batch-corrected, e.g. batch_corrected_matrix.rds)
- Homogenized study/series/sample metadata from the curated GEO microarray studies (colData, including none/genetically/pharmacologically modified classification)
- Probe-to-gene-symbol annotation used by collapseRows

## Outputs

- SummarizedExperiment objects, one per processed dataset, accessible via ExperimentHub
- Aggregated genetic and pharmacological perturbation sets assembled from the individual SummarizedExperiment objects
- Reproducibility scripts stored in the package's inst/scripts directory

## How to apply

After the curatedAdipoArray pipeline finishes (GEOquery retrieval, collapseRows gene-symbol collapsing, limma normalization, sva batch removal of data series/platform), package each processed dataset individually as an R/Bioconductor SummarizedExperiment object whose assay holds the gene-level expression matrix and whose colData carries the homogenized study/series/sample metadata (including the none/genetically/pharmacologically modified classification). Distribute access to the clean and processed SummarizedExperiment objects through ExperimentHub, and keep the generation scripts in the package's inst/scripts for reproducibility. Install the experimental data package with BiocManager and retrieve objects through ExperimentHub; the package itself contains no analysis functions, only data and documentation. The packaged objects are then ready for downstream analyses such as differential expression and gene set enrichment on the MDI-induced 3T3-L1 model.

## Related tools

- **SummarizedExperiment** (Container class used to package each processed 3T3-L1 microarray dataset individually with its expression assay and curated colData)
- **ExperimentHub** (Distribution channel providing access to the clean and processed SummarizedExperiment objects)
- **curatedAdipoArray** (Bioconductor experimental data package that documents, hosts, and exposes the packaged SummarizedExperiment objects and their generation scripts) — https://github.com/MahShaaban/curatedAdipoArray
- **BiocManager** (Installer for the curatedAdipoArray package from Bioconductor)

## Examples

```
BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- The packaged object exposes the processed expression matrix as its assay, with curated sample classification and metadata in colData, plus package metadata
- The SummarizedExperiment has the same dimensions and gene/sample order as the input normalized/batch-corrected matrix (validated downstream against batch_corrected_matrix.rds)
- Objects are retrievable through ExperimentHub after BiocManager::install("curatedAdipoArray")
- Generation scripts exist in inst/scripts so the packaged data can be reproduced, e.g. via the bcmslab/adiporeg_array Docker image

## Limitations

- curatedAdipoArray contains no R functions — it is purely a documentation-and-distribution package for the data
- Access requires Bioconductor infrastructure (BiocManager, ExperimentHub); the objects are not distributed as standalone files
- Only two of the 47 surveyed datasets were excluded (for missing probe intensities/annotations), so the packaged objects inherit the 41-dataset scope of the curation pipeline

## Evidence

- [intro] The processed datasets were packaged in `R`/`Bioconductor``SummarizedExperiment` object individually.: "The processed datasets were packaged in `R`/`Bioconductor``SummarizedExperiment` object individually."
- [intro] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] Scripts for documenting and reproducing the dataset in `inst/scripts`.: "Scripts for documenting and reproducing the dataset in `inst/scripts`."
- [readme] BiocManager::install("curatedAdipoArray"): "BiocManager::install("curatedAdipoArray")"
- [intro] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
