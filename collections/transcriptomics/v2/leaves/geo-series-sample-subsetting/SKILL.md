---
name: geo-series-sample-subsetting
description: Use when a combined multi-study dataset (multiple GEO series merged into
  one SummarizedExperiment) must be narrowed to specific studies/conditions — e.g.
  isolating Pparg-knockdown samples across GSE12929 and GSE14004 for batch-removal
  or variance analyses.
when_to_use_negative:
- The analysis requires the full curated panel (all 47 datasets / 43 studies) rather
  than specific perturbation subsets — subsetting would discard cross-study contrast
- The target study was excluded during curation for missing probe intensities or probe
  annotations — it will not be present in the SummarizedExperiment object
- Working with raw GEO data outside the curated package; use GEOquery directly instead
  of subsetting the processed object
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: curatedAdipoArray
  role: Source R/Bioconductor data package whose SummarizedExperiment object is loaded
    and subsetted by series and metadata
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment objects that
    are then filtered to the target series
- name: SummarizedExperiment
  role: Container class whose colData metadata drives the series/stage-based sample
    filtering
- name: BiocManager
  role: Installation route for the curatedAdipoArray package in R/Bioconductor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:7cf4ab1623e780ded153c76c44e72a9d474d9cc9cc4e185be17e896e504ae800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - ExperimentHub
  - R
  - R/Bioconductor
  - SummarizedExperiment
  - curatedAdipoArray
  - sva
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# geo-series-sample-subsetting

## Summary

Subset a multi-study curated GEO expression resource (e.g. the curatedAdipoArray SummarizedExperiment) down to specific data series of interest — here the two Pparg-knockdown series GSE12929 (n=48) and GSE14004 (n=9) — while retaining stage-of-differentiation metadata for downstream batch-effect and PCA analyses.

## When to use

Use when a combined multi-study dataset (multiple GEO series merged into one SummarizedExperiment) must be narrowed to specific studies/conditions — e.g. isolating Pparg-knockdown samples across GSE12929 and GSE14004 for batch-removal or variance analyses.

## When NOT to use

- The analysis requires the full curated panel (all 47 datasets / 43 studies) rather than specific perturbation subsets — subsetting would discard cross-study contrast
- The target study was excluded during curation for missing probe intensities or probe annotations — it will not be present in the SummarizedExperiment object
- Working with raw GEO data outside the curated package; use GEOquery directly instead of subsetting the processed object

## Inputs

- curatedAdipoArray SummarizedExperiment object (clean/processed, accessed via ExperimentHub)
- Homogenized colData sample metadata including data series (GSE accession), platform, and stage of differentiation

## Outputs

- Subsetted SummarizedExperiment restricted to GSE12929 (n=48) and GSE14004 (n=9) with stage-of-differentiation metadata retained
- Sample count / metadata summary confirming the subset composition

## How to apply

Load the curated resource via ExperimentHub in R/Bioconductor and access the processed SummarizedExperiment object. Use the homogenized sample metadata (colData) to filter samples to the target series — for the paper's batch-effect case study this means selecting the Pparg-knockdown series GSE12929 (n=48) and GSE14004 (n=9) and keeping the stage-of-differentiation variable intact. Because metadata were manually curated with unified language across the 43 surveyed studies, subsetting by series/stage is reliable across studies. Verify sample counts against the expected series sizes (48 and 9) before proceeding to normalization-aware steps such as PCA or sva-based batch removal. The rationale is that batch effects (data series and platform) can only be assessed and removed on a well-defined subset where both study-of-origin and biological stage are retained.

## Related tools

- **curatedAdipoArray** (Source R/Bioconductor data package whose SummarizedExperiment object is loaded and subsetted by series and metadata) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment objects that are then filtered to the target series)
- **SummarizedExperiment** (Container class whose colData metadata drives the series/stage-based sample filtering)
- **BiocManager** (Installation route for the curatedAdipoArray package in R/Bioconductor)

## Examples

```
BiocManager::install("curatedAdipoArray"); se <- curatedAdipoArray::adipo_array; sub <- se[, se$data_series %in% c("GSE12929", "GSE14004")]
```

## Evaluation signals

- Subset sample counts match the expected series sizes: GSE12929 n=48 and GSE14004 n=9 (combined n=57)
- colData of the subset retains the stage-of-differentiation variable with no NA values for retained samples
- Only samples from the two target GSE accessions remain; no samples from other series leak in
- The subset's expression matrix dimensions are consistent with the retained sample count after probe-to-gene collapse

## Limitations

- Only studies passing curation (complete probe intensities and probe annotations) are available to subset; two studies were excluded for missing probe annotations and one for missing probe intensities
- Subsetting by series assumes the unified metadata vocabulary was correctly applied during curation; misassigned series labels would propagate into the subset
- The subset is specific to microarray data series in curatedAdipoArray; it does not cover the RNA-Seq (curatedAdipoRNA) or ChIP-Seq (curatedAdipoChIP) counterparts

## Evidence

- [intro] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [abstract] The metadata were manually curated using unified language across the studies.: "The metadata were manually curated using unified language across the studies."
- [intro] One and two studies were excluded for missing probe intensities and probe annotations respectively.: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [readme] The samples are MDI-induced pre-adipocytes (3T3-L1) at different time points/stage of differentiation: "The samples are MDI-induced pre- adipocytes (3T3-L1) at different time points/stage of differentiation under"
