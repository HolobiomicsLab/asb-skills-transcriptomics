---
name: microarray-intensity-filtering
description: Use when starting from a gene-level collapsed probe-intensity matrix
  (post-collapseRows) spanning multiple microarray studies, and a comparable normalized
  expression matrix is needed before batch-effect removal or downstream differential
  expression/gene set enrichment.
when_to_use_negative:
- Input is probe-level intensities that have not yet been collapsed to gene symbols
  (run collapseRows first)
- Input is RNA-Seq or ChIP-Seq counts (curatedAdipoRNA/curatedAdipoChIP pipelines)
  rather than microarray intensities
- The matrix is already log2-transformed, filtered, and cross-study normalized (e.g.
  an existing SummarizedExperiment served via ExperimentHub)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: limma
  role: Normalizes log-transformed gene intensities across arrays and studies so intensity
    distributions are comparable
- name: curatedAdipoArray
  role: R/Bioconductor experimental data package documenting and distributing the
    curated dataset and the scripts (inst/scripts) that performed the filtering/normalization
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: R/Bioconductor
  role: Runtime environment in which the filtering, log2 transformation, and limma
    normalization are executed
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: Docker
  role: Provides the reproducible pre-processing/processing environment for reproducing
    this skill's steps
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: collapseRows
  role: Upstream step producing the gene-level collapsed intensity matrix that this
    skill filters and normalizes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:e75421c11302960b4e6b6c9b324e9d16270ebb811740af63c4d0be213ec0a9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker
  - R/Bioconductor
  - collapseRows
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# microarray-intensity-filtering

## Summary

Filter gene-level collapsed microarray intensities to drop missing and low-intensity genes (<10), log2-transform, and normalize intensities across arrays and studies with limma so distributions are comparable across the curated 3T3-L1 MDI adipocyte datasets.

## When to use

Use when starting from a gene-level collapsed probe-intensity matrix (post-collapseRows) spanning multiple microarray studies, and a comparable normalized expression matrix is needed before batch-effect removal or downstream differential expression/gene set enrichment.

## When NOT to use

- Input is probe-level intensities that have not yet been collapsed to gene symbols (run collapseRows first)
- Input is RNA-Seq or ChIP-Seq counts (curatedAdipoRNA/curatedAdipoChIP pipelines) rather than microarray intensities
- The matrix is already log2-transformed, filtered, and cross-study normalized (e.g. an existing SummarizedExperiment served via ExperimentHub)

## Inputs

- Gene-level collapsed intensity matrix (output of collapseRows probe-to-gene-symbol collapsing, obtained from GEO via GEOquery)
- Probe-to-gene-symbol annotation from the curated 3T3-L1 microarray studies

## Outputs

- normalized_gene_expression.tsv — tab-separated gene x sample matrix of limma-normalized log2 intensities with missing and low-intensity genes removed

## How to apply

Load the gene-level collapsed intensity matrix produced by the collapseRows probe-to-gene-symbol step, then remove genes with missing values or low intensity (threshold: intensity < 10) from the matrix. Log2-transform the filtered gene intensities so values are on a comparable scale. Normalize the log-transformed intensities across arrays and studies using limma so that intensity distributions are similar across arrays from different data series and platforms. Write the result (e.g. normalized_gene_expression.tsv, tab-separated) and verify the output file exists with no missing or low-intensity genes retained; the normalized matrix is then a suitable input for downstream known-batch-effect removal (data series and platform) using sva. The full pre-processing/processing environment is reproducible via the provided docker image (bcmslab/adiporeg_array), and the package ships the scripts used to generate the data in inst/scripts.

## Related tools

- **limma** (Normalizes log-transformed gene intensities across arrays and studies so intensity distributions are comparable)
- **curatedAdipoArray** (R/Bioconductor experimental data package documenting and distributing the curated dataset and the scripts (inst/scripts) that performed the filtering/normalization) — https://github.com/MahShaaban/curatedAdipoArray
- **R/Bioconductor** (Runtime environment in which the filtering, log2 transformation, and limma normalization are executed) — https://github.com/MahShaaban/curatedAdipoArray
- **Docker** (Provides the reproducible pre-processing/processing environment for reproducing this skill's steps) — https://github.com/MahShaaban/curatedAdipoArray
- **collapseRows** (Upstream step producing the gene-level collapsed intensity matrix that this skill filters and normalizes)

## Examples

```
docker pull bcmslab/adiporeg_array:latest
```

## Evaluation signals

- Output file normalized_gene_expression.tsv exists and is tab-separated with expected gene x sample dimensions
- No genes with missing values remain in the normalized matrix
- No genes with raw intensities below 10 remain after filtering
- Intensity distributions are similar across arrays/studies after limma normalization (comparable distributions across data series and platforms)
- Subsequent sva batch-effect removal (data series and platform) on the normalized matrix reproduces biological observations, per the article's batch-effect check

## Limitations

- The intensity < 10 cutoff and missing-value removal are the thresholds used for this curated 3T3-L1 dataset; re-tuning them changes the retained gene set
- Two studies were excluded for missing probe annotations and one for missing probe intensities, so filtering operates only on datasets that passed that upstream exclusion
- This skill does not remove batch effects itself — known batch effects (data series and platform) are handled downstream with sva

## Evidence

- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [abstract] Probe intensities were mapped and collapsed to genes using a reproducible pipeline.: "Probe intensities were mapped and collapsed to genes using a reproducible pipeline."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [readme] The pre-processing and processing of the data setup environment is available as a `docker` image.: "The pre-processing and processing of the data setup environment is available as a `docker` image."
