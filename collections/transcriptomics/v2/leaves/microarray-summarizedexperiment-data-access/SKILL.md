---
name: microarray-summarizedexperiment-data-access
description: Use when a task requires gene expression data from MDI-induced differentiating
  3T3-L1 pre-adipocytes under genetic (knockdown/overexpression) or pharmacological
  perturbation, and needs it loaded as a SummarizedExperiment object in R/Bioconductor.
when_to_use_negative:
- The analysis requires raw probe-level intensities or per-probe annotation from a
  specific GEO series — this skill serves the curated, probe-collapsed, batch-corrected
  gene-level objects, not raw GEO data (use GEOquery instead).
- The target data are RNA-Seq or ChIP-Seq rather than microarray — use the companion
  curatedAdipoRNA or curatedAdipoChIP packages, not curatedAdipoArray.
- The study of interest was one of the two excluded for missing probe annotations
  (or the one excluded for missing probe intensities) — its data are absent from the
  curated resource.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: curatedAdipoArray
  role: Bioconductor experimental data package that documents and distributes the
    curated 3T3-L1 microarray dataset and its curation scripts (inst/scripts)
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access mechanism for the clean and processed SummarizedExperiment data objects
- name: BiocManager
  role: Installer for the curatedAdipoArray package from Bioconductor
- name: SummarizedExperiment
  role: Container format of the processed datasets (assays, colData, metadata)
- name: Docker (bcmslab/adiporeg_array)
  role: Reproducible environment image for the data pre-processing and processing
    setup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:9da6c43910c5e75489580f1250d1d5e26b12cc9e011dd91e750605718109e9df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - Docker (bcmslab/adiporeg_array)
  - ExperimentHub
  - GO.db
  - Org.Mm.eg.db
  - R/Bioconductor
  - SummarizedExperiment
  - clusterProfiler
  - curatedAdipoArray
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# microarray-summarizedexperiment-data-access

## Summary

Retrieve the curated 3T3-L1 microarray dataset as clean and processed SummarizedExperiment objects via ExperimentHub from the curatedAdipoArray Bioconductor package, providing the standardized input for downstream differential expression and gene set enrichment analyses.

## When to use

Use when a task requires gene expression data from MDI-induced differentiating 3T3-L1 pre-adipocytes under genetic (knockdown/overexpression) or pharmacological perturbation, and needs it loaded as a SummarizedExperiment object in R/Bioconductor.

## When NOT to use

- The analysis requires raw probe-level intensities or per-probe annotation from a specific GEO series — this skill serves the curated, probe-collapsed, batch-corrected gene-level objects, not raw GEO data (use GEOquery instead).
- The target data are RNA-Seq or ChIP-Seq rather than microarray — use the companion curatedAdipoRNA or curatedAdipoChIP packages, not curatedAdipoArray.
- The study of interest was one of the two excluded for missing probe annotations (or the one excluded for missing probe intensities) — its data are absent from the curated resource.

## Inputs

- ExperimentHub resource identifier / package name (curatedAdipoArray) in an R/Bioconductor session
- BiocManager installation environment

## Outputs

- Clean and processed SummarizedExperiment objects containing curated 3T3-L1 microarray gene-level intensities, curated colData metadata, and study metadata
- R objects suitable as direct inputs for differential expression and gene set enrichment analysis

## How to apply

Install curatedAdipoArray from Bioconductor using BiocManager, then access the clean and processed data SummarizedExperiment objects through ExperimentHub rather than re-downloading raw probe intensities from GEO. The packaged data are MDI-induced 3T3-L1 microarray samples at different time points/stages of differentiation, with probe intensities already mapped and collapsed to gene symbols and metadata homogenized across 47 curated datasets from 43 studies using common vocabularies. Use the SummarizedExperiment's assay (gene-level intensities, log transformed and normalized across studies with limma), colData (unified sample metadata classifying samples as none, genetically or pharmacologically modified), and metadata slots to build the ranked gene lists or design matrices for downstream analyses. For full reproducibility of the upstream curation, the pre-processing environment is available as the Docker image bcmslab/adiporeg_array, and the original curation scripts live in inst/scripts of the package.

## Related tools

- **curatedAdipoArray** (Bioconductor experimental data package that documents and distributes the curated 3T3-L1 microarray dataset and its curation scripts (inst/scripts)) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access mechanism for the clean and processed SummarizedExperiment data objects)
- **BiocManager** (Installer for the curatedAdipoArray package from Bioconductor)
- **SummarizedExperiment** (Container format of the processed datasets (assays, colData, metadata))
- **Docker (bcmslab/adiporeg_array)** (Reproducible environment image for the data pre-processing and processing setup)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- BiocManager::install('curatedAdipoArray') completes without error and the package loads in R/Bioconductor
- ExperimentHub query returns the clean and processed SummarizedExperiment objects advertised in the package documentation
- The loaded object's colData carries curated metadata with unified vocabulary classifying samples as none, genetically or pharmacologically modified
- The assay is gene-level (probe-to-symbol collapsed) log-transformed, cross-study normalized intensities — not raw probe intensities
- Downstream steps (e.g., ranking genes by fold-change for GSEA) run directly on the object without additional probe-collapsing or batch-removal code

## Limitations

- The package contains no R functions for analysis — it only distributes data and curation scripts; all downstream analysis must use other packages
- Coverage is limited to the 43 published studies / 47 datasets that passed metadata and annotation completeness checks; two datasets were excluded (missing probe intensities/annotations)
- The processed data already have known batch effects (data series and platform) removed with sva, so users should not re-apply batch correction expecting raw values

## Evidence

- [intro] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] package purpose: "This package is for documenting and distributing a curated dataset of gene expression from MDI-induced 3T3-L1 adipocyte cell model under genetic and pharmacological modification."
- [readme] downstream use: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] installation: "The `curatedAdipoArray` package can be installed from Bioconductor using `BiocManager`."
- [intro] processing pipeline: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
