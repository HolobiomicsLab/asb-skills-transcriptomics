---
name: microarray-significance-output-gene-ranking
description: Use when you have SAM significance output from samr testing on a microarray
  matrix and need tiered top-N gene lists plus a dendrogram heatmap to test whether
  samples cluster by phenotype/response class.
when_to_use_negative:
- Raw, unnormalized microarray intensities (e.g. example_unnorm.txt before running
  Preprocess Data) — quantile normalization must be applied first
- Input lacking a SAM significance output; there are no significance scores to rank
  genes by
- Data with unresolved batch effects — PIMENTo QC assumes clusters align with response
  classes before interpreting the heatmap dendrogram
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Provides the Create Heatmap command that renders the ranked-gene heatmap with
    column dendrogram; installed from GitHub
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which PIMENTo commands (Create Heatmap on SAM output plus normalized
    matrix) are executed
- name: RStudio
  role: Open-source IDE used to run PIMENTo R commands with autocomplete
- name: Bioconductor
  role: Source of open-source packages (e.g. limma) PIMENTo leverages, including quantile
    normalization of the matrix used by the heatmap
- name: samr
  role: R package whose Significance Analysis of Microarrays code produces the significance
    output that genes are ranked from
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:1e2e3e471d02a20ba2f8c65ea77f969d5b421985f106140139777c0a8890bf5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# microarray-significance-output-gene-ranking

## Summary

Rank genes by significance score from a samr-based Significance Analysis of Microarrays output and write tiered gene lists (top 50/100/150) that feed PIMENTo's Create Heatmap command, using the quantile-normalized expression matrix to check whether samples separate by response class.

## When to use

Use when you have SAM significance output from samr testing on a microarray matrix and need tiered top-N gene lists plus a dendrogram heatmap to test whether samples cluster by phenotype/response class.

## When NOT to use

- Raw, unnormalized microarray intensities (e.g. example_unnorm.txt before running Preprocess Data) — quantile normalization must be applied first
- Input lacking a SAM significance output; there are no significance scores to rank genes by
- Data with unresolved batch effects — PIMENTo QC assumes clusters align with response classes before interpreting the heatmap dendrogram

## Inputs

- SAM significance output (samr-based Significance Analysis of Microarrays) with per-gene significance scores
- Quantile-normalized expression matrix forwarded from the PIMENTo Preprocess Data step
- Gene-list input files heatmap_input/top 50.txt, heatmap_input/top 100.txt, heatmap_input/top 150.txt

## Outputs

- Top-50-gene heatmap with samples clustering by response class (WT vs. villin-gp130Act), exported in TIFF, PDF, Postscript, and FIG formats
- Per-gene fold-change tables as CSV files
- Tiered ranked gene lists (top 50/100/150) in heatmap_input/

## How to apply

Load the SAM significance output produced by the upstream samr-based Significance Analysis of Microarrays testing step and rank genes by significance score. Write the top 50, top 100, and top 150 ranked gene lists to 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', and 'heatmap_input/top 150.txt'. In R (via RStudio), run the PIMENTo Create Heatmap command on the SAM output together with the quantile-normalized expression matrix forwarded from the Preprocess Data step. Export the top-50-gene heatmap (Fig. 8) in TIFF, PDF, Postscript, and FIG formats into heatmap_output, plus per-gene fold-change tables as CSVs. Validate that samples separate by response class — WT vs. villin-gp130Act in the mouse small-intestinal crypt data — via the column dendrogram.

## Related tools

- **PIMENTo** (Provides the Create Heatmap command that renders the ranked-gene heatmap with column dendrogram; installed from GitHub) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which PIMENTo commands (Create Heatmap on SAM output plus normalized matrix) are executed)
- **RStudio** (Open-source IDE used to run PIMENTo R commands with autocomplete)
- **Bioconductor** (Source of open-source packages (e.g. limma) PIMENTo leverages, including quantile normalization of the matrix used by the heatmap)
- **samr** (R package whose Significance Analysis of Microarrays code produces the significance output that genes are ranked from)

## Examples

```
install.packages("devtools"); source("https://bioconductor.org/biocLite.R"); biocLite(c("affy", "limma", "DESeq2")); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- heatmap_output contains the top-50 heatmap in all four graphics formats (TIFF, PDF, Postscript, FIG)
- heatmap_output contains the per-gene fold-change CSV files
- Column dendrogram of the top-50-gene heatmap separates samples into the two response classes (WT vs. villin-gp130Act)
- heatmap_input contains the three gene-list files: 'top 50.txt', 'top 100.txt', and 'top 150.txt'
- Ranking order matches the significance scores of the upstream samr output

## Limitations

- Heatmap separation of response classes depends on upstream QC: quantile normalization was determined to be the better normalization method for this data set, and no batch effect must be present
- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor), and the samr citation is an incomplete placeholder '[ref]' in the article
- PIMENTo's R Shiny user interface was not yet available in the published version — commands must be run in R via RStudio

## Evidence

- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package “samr”: [evidence span withheld — anchor status: not_found]
- [methods] PIMENTo generates a dendrogram plot of all chosen samples.: [evidence span withheld — anchor status: not_found]
- [methods] quantile normalization is the better of the two normalization methods for this data set: [evidence span withheld — anchor status: not_found]
- [methods] PIMENTo, available through Git Hub at https://www.github.com/MUSC-CGM/PIMENTo.: [evidence span withheld — anchor status: not_found]
