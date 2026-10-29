---
name: adipocyte-differentiation-stage-annotation
description: Use when working with MDI-induced 3T3-L1 microarray samples from multiple
  GEO series whose raw study-specific metadata use inconsistent vocabulary, and you
  need comparable stage-of-differentiation labels before PCA, batch correction, or
  differential expression.
when_to_use_negative:
- The samples are RNA-seq or ChIP-seq rather than the microarray (3T3-L1) data handled
  by curatedAdipoArray — use curatedAdipoRNA or curatedAdipoChIP containers instead
- The dataset already ships with manually curated metadata in a unified vocabulary,
  so re-annotation would only risk diverging from the published labels
- The goal is raw GEO download without downstream cross-study comparison, where homogenized
  stage labels add no value
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: curatedAdipoArray
  role: Source of the curated microarray SummarizedExperiment whose metadata provide
    the harmonized stage-of-differentiation annotations
  repo: github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment objects containing
    curated metadata
- name: GEOquery
  role: Used to obtain the original probe intensities, probe annotation and sample
    metadata from GEO that the curation harmonizes
- name: BiocManager
  role: Installation route for the curatedAdipoArray Bioconductor package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:bcee677518435b12855c5ee00b27997568f1120bb885dbefa299202ff3936caf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - ExperimentHub
  - GEOquery
  - R
  - R/Bioconductor
  - curatedAdipoArray
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# adipocyte-differentiation-stage-annotation

## Summary

Annotate 3T3-L1 adipocyte samples with harmonized stage-of-differentiation labels so that differentiation time point can be used as a biological variable in downstream analyses such as PCA and batch-effect assessment on curatedAdipoArray microarray data.

## When to use

Use when working with MDI-induced 3T3-L1 microarray samples from multiple GEO series whose raw study-specific metadata use inconsistent vocabulary, and you need comparable stage-of-differentiation labels before PCA, batch correction, or differential expression.

## When NOT to use

- The samples are RNA-seq or ChIP-seq rather than the microarray (3T3-L1) data handled by curatedAdipoArray — use curatedAdipoRNA or curatedAdipoChIP containers instead
- The dataset already ships with manually curated metadata in a unified vocabulary, so re-annotation would only risk diverging from the published labels
- The goal is raw GEO download without downstream cross-study comparison, where homogenized stage labels add no value

## Inputs

- SummarizedExperiment objects (clean and processed) accessed via ExperimentHub from curatedAdipoArray
- Original GEO probe intensities, probe annotations and metadata retrieved with GEOquery
- Study/series/sample metadata describing stage of differentiation (time points of MDI-induced 3T3-L1 differentiation)

## Outputs

- Homogenized colData sample-level metadata with unified stage-of-differentiation labels and modification class (none/genetic/pharmacological)
- Per-principal-component variance-explained and sample correlation table (study of origin vs stage of differentiation)
- PCA plots before and after batch removal colored/annotated by differentiation stage

## How to apply

Load the processed SummarizedExperiment objects from the curatedAdipoArray package through ExperimentHub, since these already carry manually curated metadata using unified language across the 43 surveyed studies and 47 datasets. Harmonize each sample's stage of differentiation (time points/stage of MDI-induced differentiation) to the common vocabularies the package uses to describe experimental designs, and retain sample classifications (none, genetically or pharmacologically modified). When validating the labels, compute PCA on the limma-normalized, probe-to-gene-collapsed expression matrix before and after sva empirical Bayes removal of data-series and platform batch effects: study of origin should dominate variance before correction, and stage of differentiation should correspond to PC2/PC3 after correction. This behavior confirms that the stage annotation captures true biology rather than study confounding.

## Related tools

- **curatedAdipoArray** (Source of the curated microarray SummarizedExperiment whose metadata provide the harmonized stage-of-differentiation annotations) — github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment objects containing curated metadata)
- **GEOquery** (Used to obtain the original probe intensities, probe annotation and sample metadata from GEO that the curation harmonizes)
- **BiocManager** (Installation route for the curatedAdipoArray Bioconductor package)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Stage-of-differentiation labels in colData draw on the unified vocabulary applied across studies, with no study-specific free-text stage values remaining
- Before batch removal, study of origin explains the most variance in PCA of the log-normalized expression matrix; after sva correction, stage of differentiation maps to PC2/PC3 and separates samples
- The processed SummarizedExperiment retrieved from ExperimentHub matches the package's documented colData fields and metadata structure
- Samples classify cleanly into none / genetically modified / pharmacologically modified alongside their stage annotation

## Limitations

- Two of the 47 datasets were excluded for missing probe annotations and one for missing probe intensities, so some studies cannot be annotated from this resource
- Curation quality depends on the manual unified-language mapping of heterogeneous GEO metadata; the original stage labels must still be trusted from source studies
- Stage annotation only separates samples on PCs after empirical Bayes batch removal — on raw combined data it is confounded with study of origin

## Evidence

- [abstract] Manual curation of metadata using unified language across studies: "The metadata were manually curated using unified language across the studies."
- [abstract] Batch removal validated by reproduced biological observations: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] Access to curated metadata objects via ExperimentHub: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [intro] Metadata homogenization with common vocabularies: "homogenized across studies using common vocabularies to describe the experimental designs"
- [methods] Dataset covers differentiation stages of 3T3-L1 samples: "A curated dataset of Microarrays samples. The samples are MDI-induced pre- adipocytes (3T3-L1) at different time points/stage of differentiation"
