---
name: multi-format-scientific-figure-export
description: Use when a PIMENTo figure (e.g. the top-50-gene heatmap built from the
  SAM-ranked gene list and the quantile-normalized matrix) must be exported in multiple
  graphics formats (TIFF, PDF, Postscript, FIG) into a results subdirectory.
when_to_use_negative:
- Input is raw, un-normalized microarray data (e.g. example_unnorm.txt) — it must
  first pass through the PIMENTo Preprocess Data command (quantile normalization)
  before figure generation
- Goal is statistical significance testing or ranking of genes rather than figure
  delivery — use the samr-based SAM testing step instead
- Only a single raster preview is needed and format diversity is not a deliverable
  requirement
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: PIMENTo
  role: R package whose Create Heatmap command renders and exports the top-50-gene
    heatmap in TIFF, PDF, Postscript, and FIG formats
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which the PIMENTo Create Heatmap command is executed for figure
    rendering and export
- name: RStudio
  role: Open-source IDE used to run the Create Heatmap command with autocomplete capability
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:2959b55bbe855d6fc6546472e99921e581009cdbb8992d6f7cd57689c3aa4f50
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multi-format-scientific-figure-export

## Summary

Exports PIMENTo-generated figures — such as the top-50-gene response-class heatmap — into TIFF, PDF, Postscript, and FIG formats simultaneously, so the same clustering result can be delivered in raster, vector, and editable graphics forms for publication.

## When to use

Use when a PIMENTo figure (e.g. the top-50-gene heatmap built from the SAM-ranked gene list and the quantile-normalized matrix) must be exported in multiple graphics formats (TIFF, PDF, Postscript, FIG) into a results subdirectory.

## When NOT to use

- Input is raw, un-normalized microarray data (e.g. example_unnorm.txt) — it must first pass through the PIMENTo Preprocess Data command (quantile normalization) before figure generation
- Goal is statistical significance testing or ranking of genes rather than figure delivery — use the samr-based SAM testing step instead
- Only a single raster preview is needed and format diversity is not a deliverable requirement

## Inputs

- Quantile-normalized expression matrix forwarded from the PIMENTo Preprocess Data step
- SAM significance output (samr-based Significance Analysis of Microarrays) with genes ranked by significance score
- Ranked gene-list input file, e.g. 'heatmap_input/top 50.txt' (top 50/100/150 genes)

## Outputs

- Top-50-gene heatmap exported as TIFF, PDF, Postscript, and FIG files in the heatmap_output subdirectory (Fig. 8)
- Per-gene fold-change tables written as CSV files into heatmap_output

## How to apply

Build the figure first: load the SAM significance output, rank genes by significance score, take the top-ranked gene list (e.g. the top 50 written to 'heatmap_input/top 50.txt'), and run the PIMENTo Create Heatmap command in R/RStudio on that list together with the quantile-normalized expression matrix forwarded from the Preprocess Data step. Export the rendered heatmap in all four formats (TIFF, PDF, Postscript, FIG) into the heatmap_output subdirectory, and write the per-gene fold-change tables alongside as CSVs. The rationale is that one rendering pass should yield every deliverable format — raster (TIFF) for review, vector (PDF, Postscript) for publication, and FIG for editable figures — without re-running the analysis. Validate by checking that heatmap_output holds the heatmap in all four formats plus the CSVs and that the column dendrogram separates samples by response class (WT vs. villin-gp130Act).

## Related tools

- **PIMENTo** (R package whose Create Heatmap command renders and exports the top-50-gene heatmap in TIFF, PDF, Postscript, and FIG formats) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which the PIMENTo Create Heatmap command is executed for figure rendering and export)
- **RStudio** (Open-source IDE used to run the Create Heatmap command with autocomplete capability)

## Examples

```
devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- heatmap_output contains the top-50 heatmap in all four graphics formats (TIFF, PDF, Postscript, FIG)
- Per-gene fold-change CSV files are present in heatmap_output alongside the exported figures
- The column dendrogram in the top-50 heatmap clusters samples into their response classes (WT vs. villin-gp130Act)
- The same clustering structure (dendrogram and sample ordering) is preserved identically across all exported formats

## Limitations

- Only four export formats (TIFF, PDF, Postscript, FIG) are described; other formats are not mentioned in the workflow
- Figure generation depends on upstream steps: the quantile-normalized expression matrix from Preprocess Data and the SAM-ranked gene lists must already exist
- Correctness of the separation-by-response-class claim is judged by visual inspection of the dendrogram rather than an automated metric

## Evidence

- [results] Top-50 heatmap export in four formats: [evidence span withheld — anchor status: not_found]
- [methods] Dendrogram generation in PIMENTo: [evidence span withheld — anchor status: not_found]
