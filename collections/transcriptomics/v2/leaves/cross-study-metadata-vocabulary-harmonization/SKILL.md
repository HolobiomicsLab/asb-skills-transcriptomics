---
name: cross-study-metadata-vocabulary-harmonization
description: Use when aggregating metadata from multiple public datasets (e.g., GEO
  series) whose experimental designs are described inconsistently, and you need a
  unified vocabulary to compare or pool samples across studies.
when_to_use_negative:
- The input is a single-study dataset whose metadata already uses consistent terminology
  — no cross-study harmonization is needed.
- The input is RNA-seq or ChIP-seq data (e.g., the curatedAdipoRNA or curatedAdipoChIP
  resources), which follow their own collection and pre-processing pipelines rather
  than this microarray metadata curation.
- The task is numerical processing of probe intensities (collapsing, normalization,
  batch-effect removal) rather than metadata curation.
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: GEOquery
  role: Downloads the probe intensities, probe annotations, and study/series/sample
    metadata from GEO that are then homogenized
- name: curatedAdipoArray
  role: R/Bioconductor experimental data package whose inst/scripts document the metadata
    curation and vocabulary unification
  repo: github.com/MahShaaban/curatedAdipoArray
- name: SummarizedExperiment
  role: Container carrying the harmonized sample metadata (colData) alongside each
    processed dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:8029f0de5f6def1cfc28d313ae9d141e7841bd15ab19e92943409ed12803ae93
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
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Cross-study metadata vocabulary harmonization

## Summary

Homogenizes study, series, and sample metadata harvested from multiple public GEO microarray studies (here, 3T3-L1/MDI-induced adipocyte datasets) using a unified vocabulary so samples can be consistently classified as none, genetically modified, or pharmacologically modified.

## When to use

Use when aggregating metadata from multiple public datasets (e.g., GEO series) whose experimental designs are described inconsistently, and you need a unified vocabulary to compare or pool samples across studies.

## When NOT to use

- The input is a single-study dataset whose metadata already uses consistent terminology — no cross-study harmonization is needed.
- The input is RNA-seq or ChIP-seq data (e.g., the curatedAdipoRNA or curatedAdipoChIP resources), which follow their own collection and pre-processing pipelines rather than this microarray metadata curation.
- The task is numerical processing of probe intensities (collapsing, normalization, batch-effect removal) rather than metadata curation.

## Inputs

- GEO series metadata (study, series, and sample level) downloaded via GEOquery
- Probe annotations and metadata completeness records from surveyed 3T3-L1 microarray studies

## Outputs

- Homogenized study/series/sample metadata using common vocabularies
- Sample classifications: none, genetically modified, or pharmacologically modified
- colData slot of a SummarizedExperiment object per processed dataset

## How to apply

Retrieve probe intensities, probe annotations, and metadata for each retained GEO series using GEOquery, then manually curate the study, series, and sample metadata, replacing per-study terminology with common vocabularies that describe the experimental designs. Classify each sample into one of three perturbation categories: none, genetically modified (knockdown/overexpression), or pharmacologically modified (drug treatment). Exclude datasets with incomplete metadata — the source curation dropped two of 47 examined datasets for missing probe annotations and one study for missing probe intensities. Store the harmonized sample-level metadata as colData of an R/Bioconductor SummarizedExperiment object so it travels with the processed intensities in the curatedAdipoArray-style package.

## Related tools

- **GEOquery** (Downloads the probe intensities, probe annotations, and study/series/sample metadata from GEO that are then homogenized)
- **curatedAdipoArray** (R/Bioconductor experimental data package whose inst/scripts document the metadata curation and vocabulary unification) — github.com/MahShaaban/curatedAdipoArray
- **SummarizedExperiment** (Container carrying the harmonized sample metadata (colData) alongside each processed dataset)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Every sample in the harmonized colData carries exactly one of the three classifications: none, genetically modified, or pharmacologically modified.
- Metadata terms describing experimental design use a single unified vocabulary across all included studies (no per-study synonyms remain).
- Datasets with missing probe annotations or missing probe intensities are excluded from the retained set (47 datasets examined; 41 retained).
- The curated metadata are accessible through ExperimentHub as SummarizedExperiment objects, reproducible from the scripts in inst/scripts.

## Limitations

- Metadata curation is manual and depends on the curator correctly interpreting each study's design language before mapping it to the unified vocabulary.
- Datasets with incomplete metadata (missing probe intensities or annotations) cannot be harmonized and are excluded, reducing the retained set from 47 to 41 datasets.
- The unified vocabulary in this resource covers MDI-induced 3T3-L1 differentiation designs (genetic knockdown/overexpression and drug treatments) and may not transfer directly to other cell models.

## Evidence

- [intro] Metadata homogenized with common vocabularies: "homogenized across studies using common vocabularies to describe the experimental designs"
- [abstract] Unified manual curation of metadata: "The metadata were manually curated using unified language across the studies."
- [abstract] Three-way sample classification: "Samples were classified into none, genetically or pharmacologically modified."
- [intro] Metadata completeness as exclusion criterion: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
