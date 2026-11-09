---
name: log2-transformation-of-gene-intensities
description: Use when a matrix of gene-level collapsed microarray intensities (e.g.
  from collapseRows across multiple GEO series) must be made comparable across arrays/studies
  prior to batch-effect removal or differential expression.
when_to_use_negative:
- Input is already log2-transformed and normalized (e.g. a processed SummarizedExperiment
  served via ExperimentHub) rather than raw gene-level collapsed intensities
- Working with RNA-Seq or ChIP-Seq count data (curatedAdipoRNA/curatedAdipoChIP),
  which require count-based pre-processing instead of intensity log-transformation
- Datasets excluded during curation for missing probe intensities or missing probe
  annotations, which never reach this step
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: limma
  role: normalizes the log-transformed gene intensities across arrays and studies
    so intensity distributions are comparable
- name: curatedAdipoArray
  role: R/Bioconductor experimental data package containing the scripts (inst/scripts)
    that document and reproduce the filtering, log2 transformation, and normalization
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: GEOquery
  role: downloads the probe intensities from GEO that feed the collapsing step upstream
    of this skill
- name: Docker (bcmslab/adiporeg_array)
  role: provides the pre-processing/processing environment for reproducing the transformation
    and normalization pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:2bb74757a937540078f534225fb06830781416f856dd52e5229241b18086ed54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker (bcmslab/adiporeg_array)
  - GEOquery
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# log2-transformation-of-gene-intensities

## Summary

Filter gene-level collapsed microarray intensities (drop missing values and intensities < 10), log2-transform them, and normalize across arrays/studies with limma so intensity distributions are comparable in the curated 3T3-L1 (curatedAdipoArray) datasets.

## When to use

Use when a matrix of gene-level collapsed microarray intensities (e.g. from collapseRows across multiple GEO series) must be made comparable across arrays/studies prior to batch-effect removal or differential expression.

## When NOT to use

- Input is already log2-transformed and normalized (e.g. a processed SummarizedExperiment served via ExperimentHub) rather than raw gene-level collapsed intensities
- Working with RNA-Seq or ChIP-Seq count data (curatedAdipoRNA/curatedAdipoChIP), which require count-based pre-processing instead of intensity log-transformation
- Datasets excluded during curation for missing probe intensities or missing probe annotations, which never reach this step

## Inputs

- gene-level collapsed intensity matrix (output of collapseRows probe-to-gene-symbol collapsing)
- probe intensities, annotations, and metadata downloaded from GEO via GEOquery

## Outputs

- normalized gene expression matrix (normalized_gene_expression.tsv, tab-separated, no missing or low-intensity genes retained)
- log2-transformed, limma-normalized intensities with comparable distributions across arrays/studies

## How to apply

Start from the gene-level collapsed intensity matrix produced by the probe-collapsing (collapseRows) step of the curatedAdipoArray pipeline. Remove genes with missing values or intensities below 10, then log2-transform the filtered gene intensities. Normalize the log-transformed intensities across arrays and studies using limma so intensity distributions are similar across arrays from the 43 surveyed studies. Subsequent steps (e.g. sva removal of data-series and platform batch effects) should only be applied after this normalization. Reproducibility scripts are shipped in inst/scripts of curatedAdipoArray and the environment is available as the bcmslab/adiporeg_array Docker image.

## Related tools

- **limma** (normalizes the log-transformed gene intensities across arrays and studies so intensity distributions are comparable)
- **curatedAdipoArray** (R/Bioconductor experimental data package containing the scripts (inst/scripts) that document and reproduce the filtering, log2 transformation, and normalization) — https://github.com/MahShaaban/curatedAdipoArray
- **GEOquery** (downloads the probe intensities from GEO that feed the collapsing step upstream of this skill)
- **Docker (bcmslab/adiporeg_array)** (provides the pre-processing/processing environment for reproducing the transformation and normalization pipeline)

## Examples

```
docker pull bcmslab/adiporeg_array:latest
```

## Evaluation signals

- Output file normalized_gene_expression.tsv exists and is tab-separated with the expected columns
- No genes with missing values or intensities < 10 remain in the normalized matrix
- Intensity distributions are similar across arrays after limma normalization (comparable distribution shape per array/study)
- All intensity values are on the log2 scale (no negative or raw-scale intensities present)

## Limitations

- Genes with intensities below 10 or with missing data are permanently dropped, which reduces feature space and may bias against low-abundance transcripts
- Normalization across studies does not itself remove known batch effects (data series and platform); a separate sva step is required for that
- Two datasets were excluded for missing probe annotations and one for missing probe intensities, so this step only applies to the datasets that passed curation

## Evidence

- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [full_text] Genes with intensities below 10 or with missing data were removed, and the gene-level intensities were log2-transformed and normalized using limma: [evidence span withheld — anchor status: not_found]
- [readme] The package contains the scripts that was used to generated the data in inst/scripts: "The package documents the data collection and processing. In addition to the documentation, the package contains the scripts that was
used to generated the data."
- [intro] the known batch effects (data series and platform) were removed using `sva`: "the known batch effects (data series and platform) were removed using `sva`."
