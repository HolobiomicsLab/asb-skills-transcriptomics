---
name: gene-expression-heatmap-generation-by-response-class
description: Use when you have SAM/samr significance-ranked gene lists and a quantile-normalized
  expression matrix from microarray data and need a dendrogram heatmap to check sample
  clustering by biological class.
when_to_use_negative:
- Input expression data has not yet been normalized — run PIMENTo Preprocess Data
  (quantile or LOESS normalization) first, since the heatmap expects the quantile-normalized
  matrix
- No gene-ranking step (e.g. SAM/samr significance testing) has been performed, so
  there is no significance-ordered gene list to subset into top 50/100/150
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: PIMENTo
  role: Provides the Create Heatmap command that generates the clustered heatmap and
    fold-change outputs
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Programming language in which PIMENTo and its heatmap command are run
- name: RStudio
  role: IDE used to run the Create Heatmap commands with autocomplete capability
- name: Bioconductor
  role: Source of open-source packages (e.g. affy, limma) that PIMENTo incorporates
    for upstream normalization feeding the heatmap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:bff1eb13cdec8118f1e47055a6a867199ce6eca55b5edf666309e604dc031aef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gene-expression-heatmap-generation-by-response-class

## Summary

Generate hierarchical-clustering heatmaps of the top SAM-ranked significant genes (top 50/100/150) from quantile-normalized microarray expression data using the PIMENTo Create Heatmap command, to visually confirm that samples separate by response class (e.g. WT vs. villin-gp130Act).

## When to use

Use when you have SAM/samr significance-ranked gene lists and a quantile-normalized expression matrix from microarray data and need a dendrogram heatmap to check sample clustering by biological class.

## When NOT to use

- Input expression data has not yet been normalized — run PIMENTo Preprocess Data (quantile or LOESS normalization) first, since the heatmap expects the quantile-normalized matrix
- No gene-ranking step (e.g. SAM/samr significance testing) has been performed, so there is no significance-ordered gene list to subset into top 50/100/150

## Inputs

- SAM significance output (samr-based Significance Analysis of Microarrays results) with genes ranked by significance score
- Ranked gene-list files: 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', 'heatmap_input/top 150.txt'
- Quantile-normalized expression matrix forwarded from the PIMENTo Preprocess Data step

## Outputs

- Top-50-gene heatmap with column dendrogram separating samples by response class, exported as TIFF, PDF, Postscript, and FIG files in heatmap_output
- Per-gene fold-change tables written as CSV files in heatmap_output

## How to apply

Load the significance output from the upstream samr-based Significance Analysis of Microarrays testing step and rank genes by significance score. Write the top 50, 100, and 150 ranked gene lists to files (e.g. 'heatmap_input/top 50.txt', 'top 100.txt', 'top 150.txt'). In RStudio, run the PIMENTo Create Heatmap command on the SAM output together with the quantile-normalized expression matrix forwarded from the Preprocess Data step — quantile normalization was determined to be the better normalization method for this data set. Export the top-50-gene heatmap in TIFF, PDF, Postscript, and FIG formats into heatmap_output, and write the per-gene fold-change tables as CSV files. Validate by confirming that the column dendrogram clusters samples according to response class (WT vs. villin-gp130Act), as reported for Fig. 8.

## Related tools

- **PIMENTo** (Provides the Create Heatmap command that generates the clustered heatmap and fold-change outputs) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Programming language in which PIMENTo and its heatmap command are run)
- **RStudio** (IDE used to run the Create Heatmap commands with autocomplete capability)
- **Bioconductor** (Source of open-source packages (e.g. affy, limma) that PIMENTo incorporates for upstream normalization feeding the heatmap)

## Examples

```
devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- heatmap_output contains the top-50 heatmap in all four graphics formats (TIFF, PDF, Postscript, FIG)
- heatmap_output contains the per-gene fold-change CSV files
- Samples in the top-50 heatmap separate according to response class (WT vs. villin-gp130Act), as indicated by the column dendrogram
- Ranked gene-list input files (top 50/100/150) exist in heatmap_input and correspond to the top genes by SAM significance score
- Heatmap rows correspond to the top 50 SAM-ranked genes and columns to quantile-normalized sample expression values

## Limitations

- No version numbers stated for any software dependency (limma, affy, samr, R, Bioconductor)
- PIMENTo's R Shiny user interface is not yet available; only promised in a future release
- The analysis dataset (wild-type and villin-gp130Act small intestinal crypt expression profiles) has no accession or hosting location given in the discussion section

## Evidence

- [methods] No batch effect was present in the samples: "It is evident upon inspection of Fig. 3 that there is no batch effect present in the samples, as the clusters align with the sample response classes."
- [methods] quantile normalization chosen: [evidence span withheld — anchor status: not_found]
- [discussion] samr significance testing: [evidence span withheld — anchor status: not_found]
- [readme] PIMENTo repo: "devtools::install_github("MUSC-CGM/PIMENTo")"
- [methods] dendrogram generation: [evidence span withheld — anchor status: not_found]
