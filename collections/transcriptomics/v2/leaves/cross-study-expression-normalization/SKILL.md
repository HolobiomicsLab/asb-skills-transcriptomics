---
name: cross-study-expression-normalization
description: Use when you have a gene-level collapsed intensity matrix from multiple
  microarray studies (e.g. GEO-derived 3T3-L1 arrays) and need comparable, log-scale
  normalized expression across arrays/studies.
when_to_use_negative:
- Input is raw probe-level intensities that have not yet been collapsed to gene symbols
  (collapseRows must run first)
- Input is RNA-seq count data (e.g. the curatedAdipoRNA dataset), which follows a
  different pre-processing workflow
- Goal is removing known batch effects such as data series or platform — that is a
  separate downstream step done with sva, not this normalization skill
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Normalizes the log-transformed gene intensities across arrays and studies
- name: curatedAdipoArray
  role: R/Bioconductor experimental data package containing the scripts (inst/scripts)
    that document and reproduce this filtering/transform/normalize pipeline
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: R/Bioconductor
  role: Execution environment; package installed via BiocManager, reproducible setup
    provided as a Docker image
- name: Docker
  role: Pre-processing and processing environment for reproducing the pipeline (image
    bcmslab/adiporeg_array)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:808695aca8d7d8cff46929ae8e41803f14ece8f70f0736b9e4064e8254839f50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-study-expression-normalization

## Summary

Filters missing/low-intensity genes (< 10) from the collapsed gene-level microarray intensity matrix, log2-transforms the intensities, and normalizes them across arrays and studies with limma so intensity distributions are comparable in the curated 3T3-L1 (curatedAdipoArray) datasets.

## When to use

Use when you have a gene-level collapsed intensity matrix from multiple microarray studies (e.g. GEO-derived 3T3-L1 arrays) and need comparable, log-scale normalized expression across arrays/studies.

## When NOT to use

- Input is raw probe-level intensities that have not yet been collapsed to gene symbols (collapseRows must run first)
- Input is RNA-seq count data (e.g. the curatedAdipoRNA dataset), which follows a different pre-processing workflow
- Goal is removing known batch effects such as data series or platform — that is a separate downstream step done with sva, not this normalization skill

## Inputs

- Gene-level collapsed intensity matrix (output of collapseRows probe-to-gene-symbol collapsing)
- Probe-to-gene annotation used for the upstream collapse step

## Outputs

- normalized_gene_expression.tsv — tab-separated matrix of log2-transformed, limma-normalized gene intensities with no missing or low-intensity (< 10) genes retained

## How to apply

Start from the gene-level collapsed intensity matrix produced by the probe-collapsing step (collapseRows output), not raw probe intensities. Remove genes with missing values and genes with intensities below 10 from the matrix. Log2-transform the filtered gene intensities, then normalize the log-transformed values across arrays and studies with limma so that intensity distributions are similar across arrays from different studies. Write the result to normalized_gene_expression.tsv and verify it exists in the expected tab-separated format. Note this skill ends at cross-study normalization; known batch effects (data series and platform) are removed separately with sva.

## Related tools

- **limma** (Normalizes the log-transformed gene intensities across arrays and studies)
- **curatedAdipoArray** (R/Bioconductor experimental data package containing the scripts (inst/scripts) that document and reproduce this filtering/transform/normalize pipeline) — https://github.com/MahShaaban/curatedAdipoArray
- **R/Bioconductor** (Execution environment; package installed via BiocManager, reproducible setup provided as a Docker image)
- **Docker** (Pre-processing and processing environment for reproducing the pipeline (image bcmslab/adiporeg_array))

## Examples

```
library(limma); e <- expr[apply(expr, 1, function(r) all(!is.na(r) & r >= 10)), ]; norm <- normalizeBetweenArrays(log2(e))
```

## Evaluation signals

- normalized_gene_expression.tsv exists and parses as a tab-separated matrix
- No missing values remain in the output matrix
- No genes with intensity < 10 are retained after filtering
- Values are on the log2 scale (all values non-negative raw intensities become log-scale)
- Intensity distributions are visually/quantitatively similar across arrays from different studies after normalization

## Limitations

- Two studies were excluded for missing probe intensities and missing probe annotations, so the pipeline requires complete probe annotation input
- This skill addresses cross-study normalization only; it does not remove known batch effects (data series and platform), which require a separate sva step
- The intensity threshold of 10 is specific to this curation; reusing the skill on other datasets should re-validate the cutoff

## Evidence

- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [full_text] Genes with intensities below 10 or with missing data were removed, and the gene-level intensities were log2-transformed and normalized using limma so that distributions were similar across arrays.: [evidence span withheld — anchor status: not_found]
- [intro] One and two studies were excluded for missing probe intensities and probe annotations respectively.: "One and two studies were excluded for missing probe intensities and probe annotations respectively."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [readme] the package contains the scripts that was used to generated the data.: "In addition to the documentation, the package contains the scripts that was
used to generated the data."
